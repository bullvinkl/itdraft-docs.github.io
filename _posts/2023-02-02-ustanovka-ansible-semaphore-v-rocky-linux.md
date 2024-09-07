---
title: "Установка Ansible Semaphore в Rocky Linux"
date: "2023-02-02"
categories: 
  - DevOps
tags: 
  - "ansible"
  - "ansible-tower"
  - "mariadb"
  - "nginx"
  - "ssl"
  - "postgresql"
  - "rocky-linux"
  - "semaphore"
  - "firewall"
  - "selinux"
image:
  path: /commons/merger-min.png
  alt: "Установка Ansible Semaphore"
---

> **Ansible Semaphore** — это веб-интерфейс для запуска Ansible-плейбуков с расширенными возможностями. Альтернатива Ansible Tower с открытым исходным кодом. Он позволяет запускать и управлять Ansible Tasks из веб-интерфейса.
{: .prompt-tip }

Для работы Ansible Semaphore требуется СУБД: MariaDB, BoltDB либо PostgreSQL.

## Установка MariaDB

Добавляем репозиторий MariaDB

```sh
$ curl -LsS -O https://downloads.mariadb.com/MariaDB/mariadb_repo_setup
$ sudo bash mariadb_repo_setup
```

Устанавливаем СУБД

```sh
$ sudo dnf -y install MariaDB-server MariaDB-client MariaDB-backup
```

Запускаем сервис

```sh
$ sudo systemctl enable --now mariadb
$ systemctl status mariadb
```

Запускаем скрипт инициализации и настройки

```sh
$ sudo mariadb-secure-installation
```

Создаем пользователя и базу

```sh
$ mysql -u root -p
=# CREATE DATABASE semaphoredb;
=# GRANT ALL PRIVILEGES ON semaphore.* TO 'semaphore'@'localhost' IDENTIFIED BY 'semaphorepass';
=# exit
```

## Установка PostgreSQL 15

Добавляем репозиторий PostgreSQL

```sh
$ sudo dnf -y install https://download.postgresql.org/pub/repos/yum/reporpms/EL-9-x86_64/pgdg-redhat-repo-latest.noarch.rpm
$ sudo dnf update -y
```

Отключаем модуль postgresql, что не не устанавливался PostgreSQL из дефолтных репозиториев

```sh
$ sudo dnf -qy module disable postgresql
```

Устанавливаем PostgreSQL 15

```sh
$ sudo dnf install -y postgresql15-server
```

Инициализируем БД

```sh
$ sudo /usr/pgsql-15/bin/postgresql-15-setup initdb
```

Правим конфиг `pg_hba.conf`

```sh
$ sudo nano /var/lib/pgsql/15/data/pg_hba.conf
...
host    all             all             127.0.0.1/32            md5          
# IPv6 local connections:
host    all             all             ::1/128                 md5
...
```

Запускаем PostgreSQL

```sh
$ sudo systemctl enable postgresql-15 --now
$ sudo systemctl status postgresql-15
```

Устанавливаем пароль пользователя postgres

```sh
$ sudo -u postgres psql
=# ALTER USER postgres WITH PASSWORD 'PostgreSQLPass';
```

Создаем пользователя и базу для Ansible Semaphore

```sh
=# CREATE USER semaphore WITH ENCRYPTED PASSWORD 'semaphorepass';
=# CREATE DATABASE semaphoredb OWNER semaphore;
=# GRANT ALL PRIVILEGES ON DATABASE semaphoredb TO semaphore;
=# \l
=# \q
```

## Установка Git

Устанавливаем git, смотрим версию

```sh
$ sudo dnf -y install git
$ git --version
```

## Установка Ansible Semaphore

Скачиваем дистрибутив и устанавливаем его

```sh
$ cd /tmp
$ wget https://github.com/ansible-semaphore/semaphore/releases/download/v2.8.77/semaphore_2.8.77_linux_amd64.rpm
$ sudo dnf -y install semaphore_2.8.77_linux_amd64.rpm
```

Добавляем локального пользователя от которого будет работать Ansible Semaphore и переключаемся на него

```sh
$ sudo useradd -m -d /opt/semaphore semaphore
$ sudo su - semaphore
```

Запускаем процесс установки Ansible Semaphore (для PostgreSQL)

```sh
$ semaphore setup

What database to use:
   1 - MySQL
   2 - BoltDB
   3 - PostgreSQL
 (default 1): 3

db Hostname (default 127.0.0.1:5432):
db User (default root): semaphore
db Password: semaphorepass
db Name (default semaphore): semaphoredb
Playbook path (default /tmp/semaphore): /opt/semaphore
Web root URL (optional, see https://github.com/ansible-semaphore/semaphore/wiki/Web-root-URL):
Enable email alerts? (yes/no) (default no):
Enable telegram alerts? (yes/no) (default no):
Enable slack alerts? (yes/no) (default no):
Enable LDAP authentication? (yes/no) (default no):
Config output directory (default /opt/semaphore):
...
Migrations Finished
 > Username: admin
 > Email: admin@itdraft.ru
 > Your name: Admin
 > Password: admin
```

Переключаемся на первоначального пользователя

```sh
$ exit
```

Создаем Systemd Unit

```sh
$ sudo nano /etc/systemd/system/semaphore.service
[Unit]
Description=Semaphore Ansible
Documentation=https://github.com/ansible-semaphore/semaphore
Wants=network-online.target
After=network-online.target

[Service]
Type=simple
User=semaphore
Group=semaphore
ExecReload=/bin/kill -HUP $MAINPID
ExecStart=/usr/bin/semaphore service --config=/etc/semaphore/config.json
SyslogIdentifier=semaphore
Restart=always

[Install]
WantedBy=multi-user.target
```

Создаем каталог и размещаем в нем симлинк на конфиг

```sh
$ sudo mkdir /etc/semaphore
$ sudo ln -s /opt/semaphore/config.json /etc/semaphore/config.json
```

Меняем владельца каталога

```sh
$ sudo chown -R semaphore:semaphore /etc/semaphore
```

Выполняем daemon-reload, что бы systemd нашел новые сервисы

```sh
$ sudo systemctl daemon-reload
```

Запускаем сервис

```sh
$ sudo systemctl enable --now semaphore
$ sudo systemctl status semaphore
$ sudo ss -nltup | grep 3000
```

## Установка Nginx

Добавляем репозиторий

```sh
$ sudo nano /etc/yum.repos.d/nginx.repo
[nginx]
name=Nginx Repo
baseurl=https://nginx.org/packages/rhel/$releasever/$basearch/
gpgcheck=0
enabled=1
```

Устанавливаем nginx и запускаем его

```sh
$ sudo dnf -y install nginx
$ sudo systemctl enable --now nginx
$ sudo systemctl status nginx
```

Создаем каталог для SSL сертификата

```sh
$ sudo mkdir /etc/nginx/ssl
$ cd /etc/nginx/ssl
```

Подготавливаем параметры самоподписанного SSL сертификата

```sh
$ sudo nano ssl-info.txt
[req]
default_bits       = 2048
prompt      = no
default_keyfile    = localhost.key
distinguished_name = dn
req_extensions     = req_ext
x509_extensions    = v3_ca

[ dn ]
C = RU
ST = RU
L = Russia
O = localhost
OU = Development
CN = localhost

[req_ext]
subjectAltName = @alt_names

[v3_ca]
subjectAltName = @alt_names

[alt_names]
DNS.1   = localhost
DNS.2   = 127.0.0.1
```

Создаем самоподписанный SSL сертификат

```sh
$ sudo openssl req -x509 -nodes -days 3652 -newkey rsa:2048 -keyout localhost.key -out localhost.crt -config ssl-info.txt
```

Создаем конфиг Nginx для Ansible Semaphore

```sh
$ sudo nano /etc/nginx/conf.d/semaphore.conf
upstream semaphore {
    server 127.0.0.1:3000;
}

server {
  listen 443 ssl http2;
  server_name  _;

  # add Strict-Transport-Security to prevent man in the middle attacks
  add_header Strict-Transport-Security "max-age=31536000" always;

  # SSL
  ssl_certificate /etc/nginx/ssl/localhost.crt;
  ssl_certificate_key /etc/nginx/ssl/localhost.key;

  # Recommendations from
  # https://raymii.org/s/tutorials/Strong_SSL_Security_On_nginx.html
  ssl_protocols TLSv1.1 TLSv1.2;
  ssl_ciphers 'EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH';
  ssl_prefer_server_ciphers on;
  ssl_session_cache shared:SSL:10m;

  # required to avoid HTTP 411: see Issue # 1486 
  # (https://github.com/docker/docker/issues/1486)
  chunked_transfer_encoding on;

  location / {
    proxy_pass http://semaphore/;
    proxy_set_header Host $http_host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    
    proxy_set_header X-Forwarded-Proto $scheme;

    proxy_buffering off;
    proxy_request_buffering off;
  }

  location /api/ws {
    proxy_pass http://semaphore/api/ws;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_set_header Origin "";
  }
}

server {
    # Redirect HTTP traffic to HTTPS
    listen [::]:80 ipv6only=off;
    server_name _;
    return 301 https://$host$request_uri;
}
```

Отключаем дефолтный конфиг и перезапускаем NGINX

```sh
$ sudo rm /etc/nginx/conf.d/default.conf
$ sudo nginx -t
$ sudo systemctl restart nginx
```

## Настройка Firewall и SELinux

Открываем порты и применяем правила

```sh
$ sudo firewall-cmd --permanent --add-servic={http,https}
$ sudo firewall-cmd --reload
```

Настраиваем SELinux

```sh
$ sudo setsebool -P httpd_can_network_connect 1
```
