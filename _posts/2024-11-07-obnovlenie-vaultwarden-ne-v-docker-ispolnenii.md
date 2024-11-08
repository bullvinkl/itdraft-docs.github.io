---
layout: post
title: "Обновление Vaultwarden не в Docker исполнении"
date: "2024-11-07"
categories:
  - Security-System
tags:
  - vaultwarden
  - bitwarden
  - bitwarden_rs
  - nginx
image:
  path: /commons/windows-start-menu.webp
  alt: "Обновление Vaultwarden не в Docker исполнении"
---

> **Vaultwarden** - это приложение для управления паролями, написанное на языке Rust и использующее открытое программное обеспечение. Оно является альтернативой официальному серверу Bitwarden, но более лёгким и гибким в использовании.
> Vaultwarden позволяет хранить пароли в защищенной среде, обеспечивая безопасность данных пользователей. Он поддерживает форматы KeePass и Password Safe, что позволяет использовать его совместно с другими менеджерами паролей.
{: .prompt-tip }

- [Установка Vaultwarden и PostgreSQL в Debian]({% post_url 2024-11-06-ustanovka-vaultwarden-i-postgresql-v-debian-no-docker %}){:target="_blank"} была рассмотрена в одной из предыдущих статей.
- Список статей из категории [Vaultwarden](/tags/vaultwarden/){:target="_blank"}.

Исходные данные:
- OS: `Dibian 12`
- NodeJS: `v20.18.0`
- Rust: `rustc 1.75.0 (82e1608df 2023-12-21)`
- PostgreSQL: `16`
- Домашний каталог пользователя `vaultwarden`: `/opt/vaultwarden` (`chmod 750`)
- WEB интерфейс установлен в каталог: `/opt/web-vault`

## Обновление Vaultwarden до релиза (1.30.2 > 1.32.3)

Переключаемся на пользователя `vaultwarden`
```bash
$ sudo su - vaultwarden
```

Проверяем домашний каталог
```bash
$ pwd
/opt/vaultwarden
```

Скачиваем архив и распаковываем его
```bash
$ wget https://github.com/dani-garcia/vaultwarden/archive/refs/tags/1.32.3.tar.gz
$ tar xzf 1.32.3.tar.gz
```

Переходим в каталог `vaultwarden-1.32.3` и запускаем процесс компиляции
```bash
$ cd vaultwarden-1.32.3
$ cargo clean && cargo build --features postgresql --release
```

Возвращаемся в домашний каталог и копируем конфиг
```bash
$ cd ../
$ cp vaultwarden-1.30.2/target/release/.env vaultwarden-1.32.3/target/release/.env
```

Пересоздаем симлинк
```bash
$ rm vaultwarden-latest
$ ln -s vaultwarden-1.32.3 vaultwarden-latest
```

Переключаемся на пользователя с правами `sudo` и перезапускаем сервис `vaultwarden`
```bash
$ exit
$ sudo systemctl restart vaultwarden
```

Пример моего конфига `.env`
```bash
####################
### Data folders ###
####################

## Main data folder
DATA_FOLDER=/opt/vaultwarden/data

## Web vault settings
WEB_VAULT_FOLDER=/opt/vaultwarden/web-vault/
WEB_VAULT_ENABLED=true

#########################
### Database settings ###
#########################

## Database URL
DATABASE_URL="postgresql://vaultuser:vaultpasswd@localhost/vaultdb"

## Enable WAL for the DB
ENABLE_DB_WAL=true

## Database connection retries
## Number of times to retry the database connection during startup, with 1 second delay between each retry, set to 0 to retry indefinitely
DB_CONNECTION_RETRIES=15

## Database timeout
## Timeout when acquiring database connection
DATABASE_TIMEOUT=30

## Database max connections
## Define the size of the connection pool used for connecting to the database.
DATABASE_MAX_CONNS=10

#################
### WebSocket ###
#################

## Enable websocket notifications
ENABLE_WEBSOCKET=true
#WEBSOCKET_ENABLED=true
#WEBSOCKET_ADDRESS=127.0.0.1
#WEBSOCKET_PORT=3658

########################
### General settings ###
########################

## Domain settings
DOMAIN='https://vault.itdraft.ru'

## Number of days to wait before auto-deleting a trashed item.
TRASH_AUTO_DELETE_DAYS=7

## Контролирует, разрешено ли пользователям создавать Bitwarden Sends.
## Этот параметр применяется глобально ко всем пользователям.
## Чтобы контролировать это на уровне организации, используйте политику организации «Отключить отправку».
# SENDS_ALLOWED=true

## Disable icon downloading
DISABLE_ICON_DOWNLOAD=true

## Controls if new users can register
SIGNUPS_ALLOWED=false
SIGNUPS_VERIFY=false
#SIGNUPS_DOMAINS_WHITELIST=itdraft.ru

## Controls which users can create new orgs.
## Blank or 'all' means all users can create orgs (this is the default):
ORG_CREATION_USERS=admin@itdraft.ru
## 'none' means no users can create orgs:
# ORG_CREATION_USERS=none
## A comma-separated list means only those users can create orgs:
# ORG_CREATION_USERS=admin1@example.com,admin2@example.com

## Invitations org admins to invite users, even when signups are disabled
INVITATIONS_ALLOWED=true
SHOW_PASSWORD_HINT=false

#########################
### Advanced settings ###
#########################
IP_HEADER=X-Real-IP
ICON_CACHE_TTL=86400
ICON_DOWNLOAD_TIMEOUT=10
ICON_BLACKLIST_REGEX='^(192\.168\.0\.[0-9]+|192\.168\.1\.[0-9]+)$'

## Logging to Syslog
USE_SYSLOG=true
#LOG_FILE=/var/log/vaultwarden.log
LOG_LEVEL=info

## Token for the admin interface, preferably an Argon2 PCH string
# openssl rand -base64 48
ADMIN_TOKEN='$argon2id$v=19$m=65540,t=3,p=4$IuiwAIECa99J6xElQc/A/yj7L7E7PY4hZttt5TlQFtttDBu0jFXptttOGcn446CWMUQJeKUPU4OBStjAr18'

## Enable this to bypass the admin panel security. This option is only
## meant to be used with the use of a separate auth layer in front
DISABLE_ADMIN_TOKEN=false

ADMIN_RATELIMIT_SECONDS=300
ADMIN_RATELIMIT_MAX_BURST=5
ADMIN_SESSION_LIFETIME=20

## Email 2FA settings
EMAIL_TOKEN_SIZE=6
EMAIL_EXPIRATION_TIME=600
EMAIL_ATTEMPTS_LIMIT=3

## Other MFA/2FA settings
DISABLE_2FA_REMEMBER=false

###########################
### SMTP Email settings ###
###########################

SMTP_HOST=192.168.21.16
SMTP_FROM=noreply@itdraft.ru
SMTP_FROM_NAME=Vautlwarden
SMTP_SECURITY=off
SMTP_PORT=25
#SMTP_USERNAME=
#SMTP_PASSWORD=
SMTP_TIMEOUT=15

#######################
### Rocket settings ###
#######################
#ROCKET_ENV=staging
ROCKET_ADDRESS=127.0.0.1
ROCKET_PORT=4756
```

Пример моего Systemd Unit `vaultwarden.service`
```bash
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

## Обновляем Web интерфейс

Переключаемся на пользователя `vaultwarden`
```bash
$ sudo su - vaultwarden
```

Проверяем домашний каталог
```bash
$ pwd
/opt/vaultwarden
```

Переименовываем предыдущий каталог с WEB интерфейсом
```bash
$ mv web-vault web-vault.back
```

Скачиваем архив и распаковываем его
```bash
$ wget https://github.com/dani-garcia/bw_web_builds/releases/download/v2024.6.2c/bw_web_v2024.6.2c.tar.gz
$ tar xzf bw_web_v2024.6.2c.tar.gz
```

Переключаемся на пользователя с правами `sudo` и перезапускаем сервис `nginx`
```bash
$ exit
$ sudo systemctl restart nginx
```

Пример конфига nginx
```bash
$ sudo nano /etc/nginx/conf.d/vaultwarden.conf
upstream vaultwarden-default {
  zone vaultwarden-default 64k;
  server 127.0.0.1:4756;
  keepalive 2;
}

map $http_upgrade $connection_upgrade {
    default upgrade;
    ''      "";
}

server {
  listen 80;
  listen [::]:80;
#  listen 443 ssl;
#  server_name vault.itdraft.ru;
  server_name _;

#  ssl_certificate            /etc/nginx/ssl/server.crt;
#  ssl_certificate_key        /etc/nginx/ssl/server.key;

#  if ($scheme = http) {
#      return 301 https://$server_name$request_uri;
#  }

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

# Traefik
#      allow 192.168.33.6;
#      deny all;
    }
}
```

SSL отключен, т.к. Vaultwarden проброшен через Reverse proxy
