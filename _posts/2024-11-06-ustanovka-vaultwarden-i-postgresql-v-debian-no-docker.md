---
layout: post
title: "Установка Vaultwarden и PostgreSQL в Debian 12 (не Docker)"
date: "2024-11-06"
categories:
  - Security-System
tags:
  - vaultwarden
  - bitwarden
  - bitwarden_rs
  - rust
  - nodejs
  - postgresql
  - nginx
image:
  path: /commons/computer2.jpg
  alt: "Vaultwarden и PostgreSQL в Debian 12"
---

## Подготовка

Обновляемся, устанавливаем необходимые пакеты
```bash
$ sudo apt update
$ sudo apt -y upgrade
$ sudo apt -y install git nano curl wget htop pkg-config openssl libssl3 libssl-dev
$ sudo apt -y install build-essential
```

Создаем пользователя `vaultwarden`
```bash
$ sudo useradd -m -U -r -d /opt/vaultwarden -s /bin/bash vaultwarden
$ sudo chmod 750 /opt/vaultwarden
```
## Установка Rust

Переключаемся на пользователя `vaultwarden` и устанавливаем Rust
```bash
$ sudo su - vaultwarden
$ curl --proto '=https' --tlsv1.3 -sSf https://sh.rustup.rs | sh
1) Proceed with installation (default)
```

Загружаем переменные
```bash
$ source ~/.profile
$ source ~/.cargo/env
```

Смотрим версию Rust
```bash
$ rustc -V
rustc 1.75.0 (82e1608df 2023-12-21)
```

## Установка Node JS

Переключаемся на пользователя с правами `sudo` и устанавливаем NodeJS
```bash
$ exit
$ sudo apt update && sudo apt -y install ca-certificates curl gnupg
$ curl -fsSL https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key | sudo gpg --dearmor -o /etc/apt/keyrings/nodesource.gpg
$ NODE_MAJOR=20
$ echo "deb [signed-by=/etc/apt/keyrings/nodesource.gpg] https://deb.nodesource.com/node_$NODE_MAJOR.x nodistro main" | sudo tee /etc/apt/sources.list.d/nodesource.list
$ sudo apt update && sudo apt -y install nodejs
$ which npm
/usr/bin/npm
```

## Установка PostgreSQL 16 из репозитория

Добавляем репозиторий, устанавливаем PostgreSQL
```bash
$ sudo apt -y install gnupg2
$ sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
$ wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
$ sudo apt update
$ sudo apt -y install postgresql-16 postgresql-contrib-16 libpq-dev
```

Создаем пользователя и базу в PostgreSQL
```bash
$ sudo su - postgres
$ psql
=# CREATE ROLE "vaultuser" WITH LOGIN PASSWORD 'vaultpasswd';
=# CREATE DATABASE "vaultdb" OWNER "vaultuser";
=# GRANT ALL PRIVILEGES ON DATABASE "vaultdb" TO "vaultuser";
=# GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO "vaultuser";
=# \q
$ exit
```

Редактируем конфиг `pg_hba.conf`
```bash
$ sudo nano /etc/postgresql/16/main/pg_hba.conf
...
# IPv4 local connections:
#host    all             all             127.0.0.1/32            scram-sha-256
host    all             all             127.0.0.1/32            md5
# IPv6 local connections:
#host    all             all             ::1/128                 scram-sha-256
host    all             all             ::1/128                 md5
...
```

Перезапускаем PostgreSQL
```bash
$ sudo systemctl restart postgresql@16-main.service
```

## Установка Vaultwarden

Переключаемся на пользователя `vaultwarden`, скачиваем дистрибутив, распаковываем, компилируем
``` bash
$ sudo su - vaultwarden
$ wget https://github.com/dani-garcia/vaultwarden/archive/refs/tags/1.30.1.tar.gz
$ tar xzvf 1.30.1.tar.gz
$ cd vaultwarden-1.30.1
$ cargo clean && cargo build --features postgresql --release
```

Переходим в домашний каталог, скачиваем архив с web-интерфейсом и распаковываем его
```bash
$ cd ~
$ wget https://github.com/dani-garcia/bw_web_builds/releases/download/v2024.1.2/bw_web_v2024.1.2.tar.gz
$ tar xzvf bw_web_v2024.1.2.tar.gz
```

Создаем каталог для данных
```bash
$ cd
$ mkdir data
```

Создаем пароль админа
```bash
$ ./vaultwarden hash
Password: superpassword
ADMIN_TOKEN='$argon2id$v=19$m=65540,t=3,p=4$IuiwAIECsdfsdfsdfsdfgum5TlQFdo$8NDBu0jFXpKdP9XOGcn446CWMUQJeKUPU4OBStjAr18'
```

Редактируем конфиг
```bash
$ nano /opt/vaultwarden/vaultwarden-1.30.1/target/release/.env
DATA_FOLDER=/opt/vaultwarden/data
DATABASE_URL='postgresql://vaultuser:vaultpasswd@localhost:5432/vaultdb'
DATABASE_MAX_CONNS=10
WEB_VAULT_FOLDER=/opt/vaultwarden/web-vault/
WEB_VAULT_ENABLED=true
ROCKET_ENV=staging
ROCKET_ADDRESS=127.0.0.1
ROCKET_PORT=4756
ADMIN_TOKEN='$argon2id$v=19$m=65540,t=3,p=4$IuiwAIECa99J6xElrtyrtyrtyrtyQFdo$8NDBu0jFXpKdP9XOGcn446CWMUQJeKUPU4OBStjAr18'
DISABLE_ADMIN_TOKEN=false
INVITATIONS_ALLOWED=true
DOMAIN=https://vault.itdraft.ru
#LOG_FILE=/var/log/vaultwarden.log
USE_SYSLOG=true
LOG_LEVEL=info
ENABLE_DB_WAL=true
DB_CONNECTION_RETRIES=15
ICON_CACHE_TTL=86400
DISABLE_ICON_DOWNLOAD=true
ICON_DOWNLOAD_TIMEOUT=10
ICON_BLACKLIST_REGEX='^(192\.168\.0\.[0-9]+|192\.168\.1\.[0-9]+)$'
SIGNUPS_ALLOWED=false
SIGNUPS_VERIFY=false
SIGNUPS_DOMAINS_WHITELIST=itdraft.ru,yandex.ru
#SMTP_HOST=
#SMTP_FROM=
#SMTP_FROM_NAME=
#SMTP_PORT=587
#SMTP_SSL=true
#SMTP_USERNAME=
#SMTP_PASSWORD=
#SMTP_TIMEOUT=
WEBSOCKET_ENABLED=true
WEBSOCKET_ADDRESS=127.0.0.1
WEBSOCKET_PORT=3658
IP_HEADER=X-Real-IP
ORG_CREATION_USERS=m.makarov@itdraft.ru
TRASH_AUTO_DELETE_DAYS=7
ADMIN_SESSION_LIFETIME=20
SHOW_PASSWORD_HINT=false
#https://github.com/dani-garcia/vaultwarden/blob/main/.env.template
```

Создаем симлинк
```bash
$ cd ~
$ ln -s vaultwarden-1.30.1 vaultwarden-latest
```

## Systemd Unit

Переключаемся на пользователя с правами `sudo` и создаем Systemd Unit
```bash
$ exit
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
After=network.target postgresql@16-main.service
Requires=postgresql@16-main.service

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
EnvironmentFile=-/opt/vaultwarden/vaultwarden-latest/target/release/.env
# The location of the compiled binary
ExecStart=/opt/vaultwarden/vaultwarden-latest/target/release/vaultwarden
# Set reasonable connection and process limits
LimitNOFILE=1048576
LimitNPROC=64
# Isolate bitwarden_rs from the rest of the system
PrivateTmp=true
PrivateDevices=true
ProtectHome=true
ProtectSystem=stric
# Only allow writes to the following directory and set it to the working directory (user and password data are stored here)
WorkingDirectory=/opt/vaultwarden/vaultwarden-latest/target/release/
ReadWriteDirectories=/opt/vaultwarden/vaultwarden-latest/target/release/
# Allow bitwarden_rs to bind ports in the range of 0-1024
#AmbientCapabilities=CAP_NET_BIND_SERVICE

[Install]
WantedBy=multi-user.target
```

Перечитываем Юниты, запускаем сервис Vaultwaurden и добавляем его в автозагрузку
```bash
$ sudo systemctl daemon-reload
$ sudo systemctl start vaultwarden.service
$ sudo systemctl enable vaultwarden.service
```

## Установка и настройка Nginx

Добравляем репозиторий Nginx
```bash
$ wget --quiet -O - https://nginx.org/keys/nginx_signing.key | sudo apt-key add -
$ sudo nano /etc/apt/sources.list.d/nginx.list
# NGINX repo
deb https://nginx.org/packages/mainline/debian/ bookworm nginx
deb-src https://nginx.org/packages/mainline/debian bookworm nginx
```

Устанавливаем Nginx
```bash
$ sudo apt update
$ sudo apt -y install nginx
```

Создаем конфиг для Vaultwarden
```bash
$ sudo mv /etc/nginx/conf.d/default.conf /etc/nginx/conf.d/default.conf_disabled
$ sudo nano /etc/nginx/conf.d/vaultwarden.conf
# The `upstream` directives ensure that you have a http/1.1 connection
# This enables the keepalive option and better performance
#
# Define the server IP and ports here.
upstream vaultwarden-default {
  zone vaultwarden-default 64k;
  server 127.0.0.1:4756;
  keepalive 2;
}

# Needed to support websocket connections
# See: https://nginx.org/en/docs/http/websocket.html
# Instead of "close" as stated in the above link we send an empty value.
# Else all keepalive connections will not work.
map $http_upgrade $connection_upgrade {
    default upgrade;
    ''      "";
}

server {
  listen 80;
  listen [::]:80;
  listen 443 ssl;
  server_name vault.itdraft.ru;

  ssl_certificate            /etc/nginx/ssl/server.crt;
  ssl_certificate_key        /etc/nginx/ssl/server.key;

  if ($scheme = http) {
      return 301 https://$server_name$request_uri;
  }

  error_log /var/log/nginx/vaultwarden_error.log;
  access_log /var/log/nginx/vaultwarden_access.log;

  # Allow large attachments
  client_max_body_size 128M;

    location / {
      proxy_http_version 1.1;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection $connection_upgrade;

      proxy_set_header Host $host;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header X-Forwarded-Proto $scheme;

      proxy_pass http://vaultwarden-default;
    }
}
```

Перезагружаем Nginx
```bash
$ sudo systemctl restart nginx
```

## Настройка LDAP авторизации

Переключаемся на пользователя `vaultwarden`, скачиваем дистрибутив и компилируем его
```bash
$ sudo su - vaultwarden
$ wget https://github.com/ViViDboarder/vaultwarden_ldap/archive/refs/tags/v0.6.2.tar.gz
$ tar xzf v0.6.2.tar.gz
$ cd vaultwarden_ldap-0.6.2/
$ cargo build --locked --release
```

Создаем конфиг `config.toml`
```bash
$ cd /opt/vaultwarden/vaultwarden_ldap-0.6.2/
$ cp example.config.toml target/release/config.toml
$ nano target/release/config.toml
vaultwarden_url = "https://vault.itdraft.ru"
vaultwarden_admin_token = "superpasswd"
ldap_host = "ldap.itdraft.ru"
#ldap_scheme = "ldap"
ldap_bind_dn = "cn=vaultwarden_s,ou=ServiceAccounts,dc=itdraft,dc=ru"
ldap_bind_password = "mypasswd"
ldap_search_base_dn = "ou=Users,dc=itdraft,dc=ru"
ldap_search_filter = "(&(objectClass=user)(|(memberOf=cn=staff.01,ou=Groups,dc=itdraft,dc=ru)))"
ldap_sync_interval_seconds = 3600

```

Создаем симлинк
```bash
$ cd ~
$ ln -s vaultwarden_ldap-0.6.2 vaultwarden_ldap-latest
```


Переключаемся на пользователя с правами `sudo` и создаем Systemd Unit
```bash
$ exit
$ sudo nano /etc/systemd/system/vaultwarden-ldap.service
[Unit]
Description=Bitwarden LDAP (Rust Edition)
Documentation=https://github.com/ViViDboarder/vaultwarden_ldap

After=network.target postgresql@16-main.service
Requires=vaultwarden.service

[Service]
# The user/group bitwarden_rs is run under. the working directory (see below) should allow write and read access to this user/group
User=vaultwarden
Group=vaultwarden
# The location of the .env file for configuration
EnvironmentFile=/opt/vaultwarden/vaultwarden_ldap-latest/target/release/config.toml
# The location of the compiled binary
ExecStart=/opt/vaultwarden/vaultwarden_ldap-latest/target/release/vaultwarden_ldap
# Only allow writes to the following directory and set it to the working directory (user and password data are stored here)
WorkingDirectory=/opt/vaultwarden/vaultwarden_ldap-latest/target/release/
ReadWriteDirectories=/opt/vaultwarden/vaultwarden_ldap-latest/target/release/

[Install]
WantedBy=multi-user.target
```

Перечитываем Юниты, запускаем сервис Vaultwaurden LDAP и добавляем его в автозагрузку
```bash
$ sudo systemctl daemon-reload
$ sudo systemctl start vaultwarden-ldap
$ sudo systemctl status vaultwarden-ldap
$ sudo systemctl enable vaultwarden-ldap
```

## UPD. Обновление 1.30.1 > 1.30.2

```bash
$ sudo su - vaultwarden
$ wget https://github.com/dani-garcia/vaultwarden/archive/refs/tags/1.30.2.tar.gz
$ tar xzf 1.30.2.tar.gz
$ cd vaultwarden-1.30.2
$ cargo clean && cargo build --features postgresql --release
$ cd ../
$ cp vaultwarden-1.30.1/target/release/.env vaultwarden-1.30.2/target/release/.env
$ ln -s vaultwarden-1.30.2 vaultwarden-latest
$ exit
$ sudo systemctl restart vaultwarden.service
$ sudo systemctl status vaultwarden.service
```
