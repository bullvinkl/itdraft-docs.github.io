---
title: "Установка Vaultwarden (Bitwarden_RS) и PostgreSQL не в docker-исполнении в Centos 8 / Rocky Linux"
date: "2021-10-14"
categories: 
  - Linux
  - Vaultwarden
  - PostgreSQL
tags: 
  - "bitwarden"
  - "bitwarden_rs"
  - "centos"
  - "docker"
  - "nginx"
  - "nodejs"
  - "password-manager"
  - "postgresql"
  - "rocky-linux"
  - "rust"
  - "vaultwarden"
coverImage: "computer.jpg"
image:
  path: /commons/computer.jpg
  alt: "Установка Vaultwarden"
---

> **Vaultwarden** (Bitwarden_RS) - менеджер паролей с открытым кодом. Легковесный форк широко известного Bitwarden, написан на Rust. В качестве БД используется SQLite, MariaDB, PostgreSQL
{: .prompt-tip }

## Подготовка

Обновим ОС, установим необходимые пакеты

```sh
$ sudo dnf -y install epel-release
$ sudo dnf -y update
$ sudo dnf -y install tar nano wget gcc cmake openssl-devel sqlite-devel postgresql-devel mariadb-devel
```

Добавим пользователя, от имени которого будет работать сервис Vaultwarden

```sh
$ sudo useradd -m -U -r -d /opt/vaultwarden vaultwarden
$ sudo chmod 750 /opt/vaultwarden
```

## Установка Rust

Переключимся на созданного пользователя

```sh
$ sudo su - vaultwarden
```

Установим Rust, по официальному мануалу

```sh
$ curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
1) Proceed with installation (default)
```

Настраиваем оболочку

```sh
$ source ~/.profile
$ source ~/.cargo/env
```

Проверяем

```sh
$ rustc -V
rustc 1.55.0 (c8dfcfe04 2021-09-06)
```

Переключаемся обратно на sudo-пользователя

```sh
$ exit
```

## Установка Node JS

Создадим каталог для node и установим туда node

```sh
$ sudo mkdir /opt/node
$ cd /opt/node
$ sudo wget https://nodejs.org/dist/latest-v16.x/node-v16.11.1-linux-x64.tar.xz
$ sudo tar xJvf node-v16.11.1-linux-x64.tar.xz
$ sudo ln -s /opt/node/node-v16.11.1-linux-x64 /opt/node/current
$ for f in $(ls -1 /opt/node/current/bin/); do sudo ln -s "/opt/node/current/bin/${f}" /usr/sbin/; done
```

## Установка Vaultwarden (Bitwarden_RS)

Переключаемся на созданного пользователя и устанавливаем сервис

```sh
$ sudo su - vaultwarden
$ wget https://github.com/dani-garcia/vaultwarden/archive/refs/tags/1.22.2.tar.gz
$ tar xzvf 1.22.2.tar.gz
$ cd vaultwarden-1.22.2
$ cargo build --features postgresql --release
```

Устанавливаем web-интерфейс

```sh
$ cd target/release
$ wget https://github.com/dani-garcia/bw_web_builds/releases/download/v2.23.0/bw_web_v2.23.0.tar.gz
$ tar xzvf bw_web_v2.23.0.tar.gz
```

Создадим data-директорию, которую позже укажем в настройках

```sh
$ cd
$ mkdir data
```

Создадим конфигурационный файл для Vaultwarden

```sh
$ nano /opt/vaultwarden/vaultwarden-1.22.2/target/release/.env

DATA_FOLDER=/opt/vaultwarden/data
DATABASE_URL='postgresql://pguser:pgpass@localhost:5432/vaultwarden_db'
DATABASE_MAX_CONNS=10

# openssl rand -base64 48
ADMIN_TOKEN='Q8rKXS3l6jmUYrcJGlwueZhiiIZWteGMVZe7Db/qFe0nQ68C5P5H4Bdi/1AMv4xU'
DOMAIN='https://vault.itdraft.ru'
#LOG_FILE=/var/log/vaultwarden.log
USE_SYSLOG=true
LOG_LEVEL=info
ENABLE_DB_WAL=true
DB_CONNECTION_RETRIES=15
DISABLE_ICON_DOWNLOAD=true
ICON_DOWNLOAD_TIMEOUT=10
ICON_BLACKLIST_REGEX='^(192\.168\.0\.[0-9]+|192\.168\.1\.[0-9]+)$'
#SIGNUPS_ALLOWED=false
#SIGNUPS_VERIFY=false
#SMTP_HOST=
#SMTP_FROM=
#SMTP_FROM_NAME=
#SMTP_PORT=587
#SMTP_SSL=true
#SMTP_USERNAME=
#SMTP_PASSWORD=
#SMTP_TIMEOUT=
ROCKET_ADDRESS=127.0.0.1
ROCKET_PORT=4756
WEBSOCKET_ENABLED=true
WEBSOCKET_PORT=3658
ORG_CREATION_USERS=admin
TRASH_AUTO_DELETE_DAYS=7
```

Все параметры конфигурационного файла описаны в шаблоне, из скаченного архива

Переключаемся на sudo-пользователя

```sh
$ exit
```

## Установка PostgreSQL

Отключаем postgresql из системного репозитория

```sh
$ sudo dnf -qy module disable postgresql
$ sudo dnf module list postgresql
```

Устанавливаем PostgreSQL из репозитория Postgres

```sh
$ sudo dnf -y install https://download.postgresql.org/pub/repos/yum/reporpms/EL-8-x86_64/pgdg-redhat-repo-latest.noarch.rpm
$ sudo dnf -y install postgresql13 postgresql13-server
```

Инициируем базу, запускаем сервис

```sh
$ sudo /usr/pgsql-13/bin/postgresql-13-setup initdb
$ sudo systemctl enable --now postgresql-13
$ systemctl status postgresql-13
```

Создаем пользователя БД и базу, как в конфигурационном файле

```sh
$ sudo su - postgres
$ psql
=# CREATE DATABASE vaultwarden_db;
=# \c vaultwarden_db;
=# CREATE ROLE "pguser" WITH LOGIN PASSWORD 'pgpass';
=# GRANT ALL PRIVILEGES ON DATABASE "vaultwarden_db" TO "pguser";
=# GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO "pguser";
=# \q
$ exit
```

## Systemd Unit

Создаем Systemd Unit

```sh
$ sudo nano /etc/systemd/system/vaultwarden.service

[Unit]
Description=Bitwarden Server (Rust Edition)
Documentation=https://github.com/dani-garcia/vaultwarden/
# If you use a database like mariadb,mysql or postgresql, 
# you have to add them like the following and uncomment them 
# by removing the `# ` before it. This makes sure that your 
# database server is started before bitwarden_rs ("After") and has 
# started successfully before starting bitwarden_rs ("Requires").

# Only sqlite
#After=network.target

# PostgreSQL
After=network.target mariadb.service
Requires=postgresql-13.service

# Mysql
# After=network.target mysqld.service
# Requires=mysqld.service

# PostgreSQL
# After=network.target postgresql.service
# Requires=postgresql.service

[Service]
# The user/group bitwarden_rs is run under. the working directory (see below) should allow write and read access to this user/group
User=vaultwarden
Group=vaultwarden
# The location of the .env file for configuration
EnvironmentFile=/opt/vaultwarden/vaultwarden-1.22.2/target/release/.env
# The location of the compiled binary
ExecStart=/opt/vaultwarden/vaultwarden-1.22.2/target/release/vaultwarden
# Set reasonable connection and process limits
LimitNOFILE=1048576
LimitNPROC=64
# Isolate bitwarden_rs from the rest of the system
PrivateTmp=true
PrivateDevices=true
##ProtectHome=true
ProtectSystem=strict
# Only allow writes to the following directory and set it to the working directory (user and password data are stored here)
WorkingDirectory=/opt/vaultwarden/vaultwarden-1.22.2/target/release/
ReadWriteDirectories=/opt/vaultwarden/vaultwarden-1.22.2/target/release/
# Allow bitwarden_rs to bind ports in the range of 0-1024
#AmbientCapabilities=CAP_NET_BIND_SERVICE

[Install]
WantedBy=multi-user.target

```

Запускаем сервис, проверяем

```sh
$ sudo systemctl daemon-reload
$ sudo systemctl enable vaultwarden --now
$ sudo systemctl status vaultwarden
```

## Установка и настройка Nginx

Добавляем репозиторий

```sh
$ sudo nano /etc/yum.repos.d/nginx.repo

[nginx-stable]
name=nginx stable repo
baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=1
enabled=1
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true

[nginx-mainline]
name=nginx mainline repo
baseurl=http://nginx.org/packages/mainline/centos/$releasever/$basearch/
gpgcheck=1
enabled=0
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true
```

Устанавливаем Nginx, отключаем дефолтный конфиг

```sh
$ sudo dnf -y install nginx
$ sudo mv /etc/nginx/conf.d/default.conf /etc/nginx/conf.d/default.conf_disabled
```

Создаем конфиг Nginx для Vaultwarden

```sh
$ sudo nano /etc/nginx/conf.d/vaultwarden.conf
server {
    listen 80;

    server_name vault.itdraft.ru;
    return 301 https://vault.itdraft.ru$request_uri;
}

server {
  listen 443 ssl http2;
  server_name vault.itdraft.ru;

  ssl_certificate            /etc/nginx/ssl/vault.crt;
  ssl_certificate_key        /etc/nginx/ssl/vault.key;

  error_log /var/log/nginx/vaultwarden_error.log;
  access_log /var/log/nginx/vaultwarden_access.log;

  # Allow large attachments
  client_max_body_size 128M;

  location / {
    proxy_pass http://127.0.0.1:4756;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
  }

  location /notifications/hub {
    proxy_pass http://127.0.0.1:3658;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
  }

  location /notifications/hub/negotiate {
    proxy_pass http://127.0.0.1:4756;
  }

  # Optionally add extra authentication besides the AUTH_TOKEN
  # If you don't want this, leave this part out
  location /admin {
    # See: https://docs.nginx.com/nginx/admin-guide/security-controls/configuring-http-basic-authentication/
#    auth_basic "Private";
#    auth_basic_user_file /etc/nginx/passwd/bwAdmin;

    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;

    proxy_pass http://127.0.0.1:4756;
  }
}
```

Проверяем конфиг, запускаем Nginx

```sh
$ sudo nginx -t
$ sudo systemctl enable --now nginx
```

### Настраиваем Firewall, отключаем SELinux

Открываем web-порты

```sh
$ sudo firewall-cmd --permanent --add-service=http
$ sudo firewall-cmd --permanent --add-service=https
$ sudo firewall-cmd --reload
```

Отключаем SELinux

```sh
$ sudo nano /etc/sysconfig/selinux
SELINUX=disabled

$ sudo setenforce 0
```

Установка завершена.  
Переходим в браузере по нужному URL, создаем пользователя, экспортируем пароли
