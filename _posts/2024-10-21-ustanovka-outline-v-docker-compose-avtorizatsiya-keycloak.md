---
layout: post
title: "Установка Outline в Docker Compose, авторизация Keycloak"
date: "2024-10-21"
categories:
  - Content-Management
tags:
  - wiki
  - docker-compose
  - keycloak
  - reverse-proxy
image:
  path: /commons/pexels-lukas-574071-scaled.jpg
  alt: "Установка Outline в Docker Compose, авторизация Keycloak"
---

> **Outline** (Team knowledge base & wiki) - это open-source платформа для создания и управления внутренними документами, спецификациями продукта, ответами на вопросы поддержки, заметками встреч, руководством по настройке и другими типами контента для команды. Она предназначена для организации знаний и информации внутри команды, обеспечивая доступ к ней для коллег и уменьшая повторяющиеся запросы.
{: .prompt-tip }

## Установка Docker и Docker Compose

- [Установка Docker и Docker Compose в Debian]({% post_url 2022-11-15-svyazka-wordpress-v-docker-subd-lokalno-v-debian-11 %}) была рассмотрена в одной из предыдущих статей.
- [Установка Docker и Docker Compose в RHEL-like дистрибутиве]({% post_url 2023-02-07-docker-private-registry-ssl-s-avtorizaciej-i-portainer-v-rocky-linux %}) так же была рассмотрена в одной из предыдущих статей.

## Настройка Keycloak

- [Установка Keycloak + PostgeSQL в Linux]({% post_url 2022-08-29-ustanovka-keycloak-i-postgesql-v-linux-centos-rocky-debian %}){:target="_blank"} была рассмотрена в одной из предыдущих статей.
- Список статей из категории [Keycloak](/tags/keycloak/){:target="_blank"}

В Web интерфейсе Keycloak создаем нового клиента

![Images](/assets/img/posts/2024/10/21/outline-kc1.png){: w="300" }
_Настройки Keycloak_

- Client ID: `outline`
- Name: `outline`
- Root URL: `https://docs.itdraft.ru`
- Home URL: `https://docs.itdraft.ru`
- Valid redirect URIs: `https://docs.itdraft.ru/auth/oidc.callback`
- Web origins: `https://docs.itdraft.ru`

![Images](/assets/img/posts/2024/10/21/outline-kc2.png){: w="300" }
_Настройки Keycloak_

- Client authentication: `On`
- Authentication flow: 
  - Standard flow: `On`
  - Direct access grants: `On`

Копируем `Client secret` в настройках клиента `outline` (во вкладке `Credentials`)

## Установка Outline

Создадим каталог `/opt/outline` и назначим текущего пользователя владельцем каталога

```sh
$ sudo mkdir -p /opt/outline
$ sudo chown -R $(whoami):$(whoami) /opt/outline
```

Перейдем в каталог и создадим файл `docker-compose.yml` с содержимым

```sh
$ cd /opt/outline
$ nano docker-compose.yml
```

```yaml
services:
  outline:
    # image: outlinewiki/outline:latest
    image: flameshikari/outline-ru:0.80.2
    restart: always
    env_file: ./docker.env
    # command: sh -c "yarn --env=production-ssl-disabled && yarn start --env=production-ssl-disabled"
    ports:
      - "80:3000"
    volumes:
      - storage-data:/var/lib/outline/data
    depends_on:
      - postgres
      - redis

  redis:
    image: redis:7
    restart: always
    env_file: ./docker.env
    expose:
      - "6379"
    volumes:
      - ./redis.conf:/usr/local/etc/redis.conf
    command: ["redis-server", "/usr/local/etc/redis.conf"]
    healthcheck:
      test: [ "CMD", "redis-cli", "ping" ]
      interval: 10s
      timeout: 30s
      retries: 3

  postgres:
    image: postgres:16
    restart: always
    env_file: ./docker.env
    expose:
      - "5432"
    volumes:
      - database-data:/var/lib/postgresql/data
    healthcheck:
      test: [ "CMD", "pg_isready", "-d", "outline", "-U", "outline_user" ]
      interval: 30s
      timeout: 20s
      retries: 3

volumes:
  storage-data:
  database-data:
```

`image: flameshikari/outline-ru:0.80.2`  - в данном примере используется образ с руссификацией

Создадим файл `docker.env`

```sh
$ sudo nano docker.env

## Required Variables
NODE_ENV=production
SECRET_KEY=%SECRET_KEY%    # openssl rand -hex 32
UTILS_SECRET=%UTILS_SECRET%    # openssl rand -hex 32
FORCE_HTTPS=false
ENABLE_UPDATES=true
WEB_CONCURRENCY=2
 
## Postgres Variables
POSTGRES_USER=outline_user
POSTGRES_PASSWORD=%POSTGRES_PASSWORD%    # openssl rand -hex 32
POSTGRES_DB=outline
DATABASE_URL=postgres://outline_user:${POSTGRES_PASSWORD}@postgres:5432/outline
PGSSLMODE=disable
 
## Redis Variables
REDIS_URL=redis://redis:6379
 
## Domain Variables
URL=https://docs.itdraft.ru
PORT=3000

# The default interface language
DEFAULT_LANGUAGE=ru_RU
 
## Rate Limiting Variables
RATE_LIMITER_ENABLED=true
RATE_LIMITER_DURATION_WINDOW=60
RATE_LIMITER_REQUESTS=1000
 
## Local File Storage Variables - IF USING AWS, CHANGE VALUE TO s3
FILE_STORAGE=local
FILE_STORAGE_LOCAL_ROOT_DIR=/var/lib/outline/data
FILE_STORAGE_UPLOAD_MAX_SIZE=26214400

## OIDC Authentication Variables
# You will need to get this from the Keycloak admin panel. Please see "Keycloak OIDC Setup".
OIDC_CLIENT_ID=outline
OIDC_CLIENT_SECRET=%OIDC_CLIENT_SECRET%    # From Keycloak
OIDC_AUTH_URI=https://keycloak.itdraft.ru:8443/realms/itdraft/protocol/openid-connect/auth
OIDC_TOKEN_URI=https://keycloak.itdraft.ru:8443/realms/itdraft/protocol/openid-connect/token
OIDC_USERINFO_URI=https://keycloak.itdraft.ru:8443/realms/itdraft/protocol/openid-connect/userinfo
## Changes what shows up on the login button.
OIDC_DISPLAY_NAME=Keycloak
## Probably shouldn't change
OIDC_USERNAME_CLAIM=preferred_username
OIDC_SCOPES=openid profile email

# LOG_LEVEL=info
```

В данном файле значения параметров `SECRET_KEY`, `UTILS_SECRET` и `POSTGRES_PASSWORD` получаем в результате выполнения команды `openssl rand -hex 32`. Значение параметра `OIDC_CLIENT_SECRET` мы берем после добавления клиента Keycloak

> Пример файла настроек из [официального репозитория](https://github.com/outline/outline/blob/main/.env.sample){:target="_blank"}
{: .prompt-info }

Настраиваем сетевую связанность Outline и Keycloak, что бы заработала авторизация

## Настройка Reverse Proxy

Осталось настроить Reverse Proxy на 80 порт сервер с Outline. Как вариант можно поднять Nginx (традиционным способом или прописать сервис в docker compose)

Пример конфига (не проверял):

```bash
sudo nano /etc/nginx/conf.d/outline.conf

server {
    listen 80;
    listen [::]:80;
 
    server_name docs.itdraft.ru;
 
    return 301 https://$host$request_uri;
}
 
server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
 
    server_name docs.itdraft.ru;
 
    access_log off;
    error_log /var/log/nginx/outline-error.log;
 
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA:ECDHE-RSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-RSA-AES256-SHA256:DHE-RSA-AES256-SHA:ECDHE-ECDSA-DES-CBC3-SHA:ECDHE-RSA-DES-CBC3-SHA:EDH-RSA-DES-CBC3-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:DES-CBC3-SHA:!DSS;
    ssl_prefer_server_ciphers on;
    ssl_session_timeout 24h;
    ssl_stapling on;
    ssl_stapling_verify on;
    ssl_certificate /etc/letsencrypt/live/docs.itdraft.ru/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/docs.itdraft.ru/privkey.pem;
    ssl_trusted_certificate /etc/letsencrypt/live/docs.itdraft.ru/chain.pem;
    ssl_dhparam /etc/ssl/certs/dhparam.pem;
    ssl_session_cache shared:OutlineSSL:10m;
    ssl_session_tickets off;
 
    location / {
        proxy_pass http://127.0.0.1:80;
         
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "Upgrade";
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Scheme $scheme;
        proxy_set_header X-Forwarded-Proto $scheme;
 
        proxy_redirect off;
    }
}
```
