---
title: "Docker Private Registry SSL с авторизацией и Portainer в Rocky Linux"
date: "2023-02-07"
categories: 
  - Linux
  - Docker
tags: 
  - "docker"
  - "docker-compose"
  - "docker-registry"
  - "portainer"
  - "rocky-linux"
  - "ssl"
image:
  path: /commons/recommendation_sys-min.png
  alt: "Docker Private Registry SSL с авторизацией и Portainer"
---

> **Portainer** — UI для управления Docker контейнерами из браузера. Проект с открытым исходным кодом.  
> **Docker Private Registry** - приватный репозиторий для docker-контейнеров.

## Установка Docker

Устанавливаем Docker-CE

```sh
$ sudo dnf -y install docker-ce --nobest
```

Добавляем нашего пользователя, под которым настраиваем ОС, в группу Docker

```sh
$ sudo usermod -aG docker $(whoami)
```

Применяем изменения к группам

```sh
$ newgrp docker
```

Добавляем сервис в автозагрузку и запускаем его

```sh
$ sudo systemctl enable --now docker
```

## Установка Docker Compose

Скачиваем docker-compose

```sh
$ sudo curl -L https://github.com/docker/compose/releases/download/v2.15.1/docker-compose-linux-x86_64 -o /usr/local/bin/docker-compose
```

Делаем файл исполняемым и создаем симлинк

```sh
$ sudo chmod +x /usr/local/bin/docker-compose
$ sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```

## Настраиваем Firewall

Для запуска docker compose с внешним доступом к веб-серверу необходимо включить NAT

```sh
$ sudo firewall-cmd --zone=public --add-masquerade --permanent
$ sudo firewall-cmd --reload
```

## Подготовка к установке Docker Private Registry

Создаем необходимые каталоги и меняем владельца

```sh
$ sudo mkdir -p /opt/docker/registry/{certs,data,auth}
$ sudo chown -R $(whoami):$(whoami) /opt/docker/registry
```

Переходим в каталог certs и генерим самоподписанный сертификат

```sh
$ cd /opt/docker/registry/certs
$ openssl req -batch -new -newkey rsa:2048 -nodes -keyout registry.key -subj '/C=RU/ST=Moscow/L=Moscow/O=Company/OU=IT/CN=registry/emailAddress=admin@itdraft.ru' -out registry.csr
$ openssl x509 -in registry.csr -out registry.crt -req -signkey registry.key -days 365
```

## Настройка авторизации

Устанавливаем набор утилит

```sh
$ sudo dnf -y install httpd-tools
```

Создаем связку логин:пароль

```sh
$ htpasswd -Bbc /opt/docker/registry/auth/.htpasswd admin pass
```

## Настройка Docker Private Registry и запуск через Docker Composer

Создаем docker-compose файл

```sh
$ nano /opt/docker/registry/docker-compose.yml 
version: '3.5'
services:
  registry:
    container_name: registry
    hostname: 'localhost'
    image: registry:latest
    environment:
      - REGISTRY_LOG_LEVEL=info
      - REGISTRY_STORAGE_FILESYSTEM_ROOTDIRECTORY=/var/lib/registry
      - REGISTRY_HTTP_ADDR=0.0.0.0:443
      - REGISTRY_HTTP_TLS_CERTIFICATE=/opt/certs/registry.crt
      - REGISTRY_HTTP_TLS_KEY=/opt/certs/registry.key
      - REGISTRY_AUTH=htpasswd
      - REGISTRY_AUTH_HTPASSWD_PATH=/opt/auth/.htpasswd
      - REGISTRY_AUTH_HTPASSWD_REALM="Registry Realm"
    volumes:
      - '/opt/docker/registry/data:/var/lib/registry'
      - '/opt/docker/registry/certs:/opt/certs'
      - '/opt/docker/registry/auth:/opt/auth'
    ports:
      - '5443:443'
    restart: unless-stopped
```

Запускаем

```sh
$ cd /opt/docker/registry
$ docker-compose up -d
```

Открываем порт

```sh
$ sudo firewall-cmd --add-port=5443/tcp --permanent
$ sudo firewall-cmd --reload
```

Пробуем авторизоваться

```sh
$ docker login -u admin https://localhost:5443
Login Succeeded

$ docker logout https://localhost:5443
```

## Настройка Portainer и запуск через Docker Composer

Создаем каталог и меняем владельца

```sh
$ sudo mkdir /opt/docker/portainer
$ sudo chown -R $(whoami):$(whoami) /opt/docker/portainer
```

Создаем docker-compose файл

```sh
$ nano /opt/docker/portainer/docker-compose.yml
version: '3.9'
services:
  portainer:
    image: portainer/portainer-ce:latest
    container_name: portainer
    security_opt:
      - no-new-privileges:true
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - ./portainer-data:/data
    ports:
      - 9000:9000
    restart: unless-stopped
```

Запускаем

```sh
$ cd /opt/docker/portainer
$ docker-compose up -d
```

Открываем порт

```sh
$ sudo firewall-cmd --add-port=9000/tcp --permanent
$ sudo firewall-cmd --reload
```

Авторизуемся, задаем пароль

```
http://localhost:9000/
```