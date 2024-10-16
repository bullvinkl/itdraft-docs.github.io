---
layout: post
title: "Установка Wiki.js в Docker Compose и подключение Elasticsearch, Keycloak, Git"
date: "2024-10-16"
categories:
  - DevOps
  - Development
tags:
  - wiki.js
  - docker-compose
  - elasticsearch
  - keycloak
  - git
image:
  path: /commons/89df389c8b4e91e605d775dc0f7f85ea.jpg
  alt: "Установка Wiki.js как Docker Compose и подключение Elasticsearch, Keycloak, Git"
---

> **Wiki.js** - это вики-движок с открытым исходным кодом, написанный на языке JavaScript и использующий Node.js как основу. Он позволяет создавать и управлять веб-вики, использующими Markdown для форматирования текста.
> Wiki.js предназначен для создания и управления централизованными базами знаний, документацией, информационными ресурсами и другими типами контента.
{: .prompt-tip }

## Установка Docker и Docker Compose

- [Устанавливаем Docker и Docker Compose в Debian]({% post_url 2022-11-15-svyazka-wordpress-v-docker-subd-lokalno-v-debian-11 %}) была рассмотрена в одной из предыдущих статей.
- [Устанавливаем Docker и Docker Compose в RHEL-like дистрибутиве]({% post_url 2023-02-07-docker-private-registry-ssl-s-avtorizaciej-i-portainer-v-rocky-linux %}) так же была рассмотрена в одной из предыдущих статей.

## Установка Wiki.js

Создадим каталог `/opt/wikijs` и назначим текущего пользователя владельцем каталога

```sh
$ sudo mkdir -p /opt/wikijs
$ sudo chown -R $(whoami):$(whoami) /opt/wikijs
```

Перейдем в каталог и создадим файл `docker-compose.yml` с содержимым

```sh
$ cd /opt/wikijs
$ nano docker-compose.yml
```

```yaml
services:

  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: wiki
      POSTGRES_PASSWORD: wikijsrocks
      POSTGRES_USER: wikijs
    logging:
      driver: "none"
    restart: unless-stopped
    volumes:
      - db-data:/var/lib/postgresql/data

  wiki:
    image: ghcr.io/requarks/wiki:2
    depends_on:
      - db
    environment:
      DB_TYPE: postgres
      DB_HOST: db
      DB_PORT: 5432
      DB_USER: wikijs
      DB_PASS: wikijsrocks
      DB_NAME: wiki
    restart: unless-stopped
    ports:
      - "80:3000"

  elasticsearch:
    image: elasticsearch:7.17.6
    restart: unless-stopped
    environment:
      discovery.type: single-node
      ES_JAVA_OPTS: "-Xms2G -Xmx2G"

volumes:
  db-data:
```

Запускаем Wiki.js

```sh
$ docker compose up -d
```

Переходим в Web-интерфейс, задаем email и пароль для административного доступа

## Подключаем Elasticsearch
> Elasticsearch - это масштабируемая поисковая система, которая позволяет эффективно индексировать и обрабатывать большие объемы данных, включая текст, числа, даты и другие типы данных. Она написана на Java и использует JSON REST API для работы с данными, а также основана на технологии Apache Lucene для полнотекстового поиска.
{: .prompt-tip }

Как видно из содержимого файла `docker-compose.yml`, контейнер Elasticsearch мы уже подключили.

Для активации Elasticsearch переходим

- `Администрирование > Поисковая система`

Прописываем следующие настройки

![Images](/assets/img/posts/2024/10/16/wikijs-elastic.png){: w="300" }
_Настройки Elasticsearch_

- Elasticsearch Version: `7.x`
- Host: `http://elasticsearch:9200`
- Verify TLS Certificate: `Off`
- Index Name: `wiki`
- Analyzer: `simple`
- Sniff on start: `Off`
- Sniff Interval: `0`

Поиск Elasticsearch подключен, можно проверять

## Подключаем авторизацию Keycloak

На стороне Keycloak сервера создаем клиента со следующими параметрами

![Images](/assets/img/posts/2024/10/16/wikijs-keycloak1.png){: w="300" }
_Настройки Keycloak_

- Client ID: `wikijs`
- Root URL: `https://wiki.itdraft.ru`
- Home URL: `https://wiki.itdraft.ru`
- Valid redirect URIs: `/login/122b715d-1111-4a0b-a232-6f3503434b46/callback`  - `# этот параметр берется из админки Wiki.js`
- Valid redirect URIs: `https://wiki.itdraft.ru/*`
- Web origins: `*`

Копируем `Client secret` в настройках клиента `wikijs` во вкладке Credentials

Переходим в админку Wiki.js, добавляем авторизацию Keycloak

- `Администрирование > Авторизация > Добавить стратегию > Keycloak`

![Images](/assets/img/posts/2024/10/16/wikijs-keycloak2.png){: w="300" }
_Настройки Keycloak в Wiki.js_

- Host: `https://keycloak.itdraft.ru:8443`
- Realm: `itdraft`
- Client ID: `wikijs`
- Client Secret: `###`
- Authorization Endpoint URL: `https://keycloak.itdraft.ru:8443/realms/itdraft/protocol/openid-connect/auth`
- Token Endpoint URL: `https://keycloak.itdraft.ru:8443/realms/itdraft/protocol/openid-connect/token`
- User Info Endpoint URL: `https://keycloak.itdraft.ru:8443/realms/itdraft/protocol/openid-connect/userinfo`

![Images](/assets/img/posts/2024/10/16/wikijs-keycloak3.png){: w="300" }
_Настройки Keycloak в Wiki.js_

- Logout Endpoint URL: `https://keycloak.itdraft.ru:8443/realms/itdraft/protocol/openid-connect/logout`
- Разрешить самостоятельную регистрацию: `On`
- Ограничить указанными почтовыми доменами: `itdraft.ru`
- Назначить в группу: `IT`  - `# данную группу необходимо создать заранее`
- В `Справке по конфигурации` берем значение `Callback URL / Redirect URI`, которое прописываем на стороне Keycloak

Настройка авторизации Keycloak завершена, можно проверять

## Подключаем хранилище Git

В нашей системе управления репозиториями (GitHub, Gitlab) создаем репозиторий для хранения документации Wiki.js

Создаем приватный и публичный ключ, для подключения к репозиторию

```sh
$ ssh-keygen -f wikijs -t ed25519 -C "wikijs-ed25519-key"
```

Переходим в админку Wiki.js, добавляем хранилище Git

- `Администрирование > Хранилище > Git`

![Images](/assets/img/posts/2024/10/16/wikijs-git1.png){: w="300" }
_Настройки Git в Wiki.js_

- Authentication Type: `ssh`
- Repository URI: `ssh://git@gitlab.itdraft.ru/m.makarov/wikijs.git`
- Branch: `main`
- SSH Private Key Mode: `contents`
- B - SSH Private Key Contents: `###` - `# Приватный ключ, который мы генерировали ранее`

![Images](/assets/img/posts/2024/10/16/wikijs-git2.png){: w="300" }
_Настройки Git в Wiki.js_

- Default Author Email: `m.makarov@itdraft.ru`
- Default Author Name: `Максим Макаров`
- Способ синхронизации: `Двунаправленная синхронизация`

Подключение хранилища Git закончено
