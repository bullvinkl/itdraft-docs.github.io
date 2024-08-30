---
layout: post
title: "Связка WordPress и Docker, локальная MariaDB в Debian 11"
date: "2022-11-15"
categories:
  - Linux
  - Docker
tags:
  - linux
  - docker
  - docker-compose
  - debian
  - mysql
  - mariadb
image:
  path: /commons/mockup-2443050_1280.jpg
  alt: "Связка WordPress и Docker, локальная MariaDB в Debian 11"
---

> **Docker** — программное обеспечение для автоматизации развёртывания и управления приложениями в средах с поддержкой контейнеризации, контейнеризатор приложений.

Использование СУБД в Docker исполнении в проде - не очень хорошая идея. По-этому решил настроить один из боевых серверов c WordPress в Docker исполнении, но MySQL оставить локально

## Установка Docker из репозитория

Устанавливаем необходимые пакеты, создаем директорию и скачиваем цифровую подпись
```sh
$ sudo apt -y install ca-certificates curl gnupg lsb-release
$ sudo mkdir -p /etc/apt/keyrings
$ curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

Добавляем репозиторий в систему
```sh
$  echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

$ sudo chmod a+r /etc/apt/keyrings/docker.gpg
$ sudo apt update
```

Устанавливаем дистрибутив
```sh
$ sudo apt -y install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

Добавляем нашего пользователя в группу docker
```sh
$ sudo usermod -aG docker $(whoami)$ newgrp docker
```

## Установка Docker Compose

Скачиваем дистрибутив, файл исполняемым и создаем симлинк
```sh
$ sudo curl -L "https://github.com/docker/compose/releases/download/v2.12.2/docker-compose-linux-x86_64" -o /usr/local/bin/docker-compose

$ sudo chmod +x /usr/local/bin/docker-compose
$ sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```

Проверяем
```
$ docker-compose -v
```

## Установка MariaDB

Устанавливаем дистрибутив, запускаем первоначальную настройку MariaDB
```sh
$ sudo apt update
$ sudo apt -y install mariadb-server mariadb-client
$ sudo mysql_secure_installation
```

Создаем пользователя и базу
```sh
$ sudo mariadb
> CREATE DATABASE wp_mydatabase;
> CREATE USER 'wp_myuser'@'%' IDENTIFIED BY 'sercretpasswd';
> GRANT ALL PRIVILEGES ON wp_mydatabase.* to wp_myuser@'%';
> FLUSH PRIVILEGES;
> exit
```

Проверяем
```sh
$ sudo mariadb
> SELECT User, Host FROM mysql.user;
+-------------+-----------+
| User        | Host      |
+-------------+-----------+
| wp_myuser   | %         |
| mariadb.sys | localhost |
| mysql       | localhost |
| root        | localhost |
+-------------+-----------+
```

Настраиваем MySQL, иначе контейнер не сможет подключиться к БД
```sh
$ sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
...
#bind-address            = 127.0.0.1
bind-address            = 172.17.0.1

$ sudo systemctl restart mysql
```

## Пример проекта

```sh
$ cat ~/proj/docker-compose.yml
version: '3'

services:
  wp:
    image: wordpress:php8.1-fpm-alpine
    container_name: wordpress
    user: 1000:1000
    environment:
      WORDPRESS_DB_HOST: ${MYSQL_HOST}
      WORDPRESS_DB_NAME: ${MYSQL_DATABASE}
      WORDPRESS_DB_USER: ${MYSQL_USER}
      WORDPRESS_DB_PASSWORD: ${MYSQL_PASSWORD}
      WORDPRESS_TABLE_PREFIX: ${WORDPRESS_TABLE_PREFIX}
    volumes:
      - ./data:/var/www/html
      - ./php/wordpress.ini:/usr/local/etc/php/conf.d/wordpress.ini
    restart: unless-stopped

  nginx:
    image: nginx:alpine
    container_name: nginx
    volumes:
      - ./nginx:/etc/nginx/conf.d
      - ./data:/var/www/html
      - ./ssl:/etc/nginx/ssl
    ports:
      - '80:80'
      - '443:443'
    depends_on:
      - wp
    restart: unless-stopped
```

```sh
$ cat ~/proj/php/wordpress.ini
file_uploads = On
memory_limit = 256M
upload_max_filesize = 64M
post_max_size = 64M
max_execution_time = 600
max_input_time = 1000
max_input_vars = 3000
```

```sh
$ cat ~/proj/nginx/wp.conf
server {
    listen 80 default_server;
    server_name _;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name $SITE_URL;

    root /var/www/html;
    index index.php;

    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;

    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Host $host;

    client_max_body_size 64m;
    client_body_buffer_size 4M;

    ssl_certificate           /etc/nginx/ssl/fullchain.pem;
    ssl_certificate_key       /etc/nginx/ssl/privkey.pem;
    ssl_trusted_certificate   /etc/nginx/ssl/chain.pem;

    location = /favicon.ico {
        log_not_found off;
        access_log off;
    }

    location = /robots.txt {
        allow all;
        log_not_found off;
        access_log off;
    }

    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass wp:9000;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
    }
}
```

```sh
$ cat ~/proj/.env
# Root password for your database
MYSQL_ROOT_PASSWORD=myrootpasswd

# Database name, user and password for your wordpress
MYSQL_HOST=172.17.0.1:3306
MYSQL_DATABASE=wp_mydatabase
MYSQL_USER=wp_myuser
MYSQL_PASSWORD=sercretpasswd

# Table prefix
WORDPRESS_TABLE_PREFIX=wp_

# Site URL
SITE_URL=itdraft.ru
```