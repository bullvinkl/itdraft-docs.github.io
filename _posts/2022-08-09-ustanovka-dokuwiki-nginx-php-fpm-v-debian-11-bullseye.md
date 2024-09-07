---
title: "Установка DokuWiki + Nginx + PHP-FPM в Debian 11 Bullseye"
date: "2022-08-09"
categories: 
  - Linux
  - Debian
  - DokuWiki
tags: 
  - "debian"
  - "dokuwiki"
  - "linux"
  - "nginx"
  - "php-fpm"
image:
  path: /commons/laptop-2.jpg
  alt: "Установка DokuWiki + Nginx + PHP-FPM в Debian"
---

> **DokuWiki** — простой, но достаточно мощный вики-движок, который может быть использован для создания любой документации. Она ориентирована на команды разработчиков, рабочие группы и небольшие компании. Все данные хранятся в простых текстовых файлах, поэтому для работы не требуется СУБД
{: .prompt-tip }

## Подготовка

Обновляемся

```sh
$ sudo apt update && sudo apt -y upgrade
```

Устанавливаем софт (мой стандартный набор)

```sh
$ sudo apt -y install nano curl bind9-utils telnet wget net-tools traceroute git tcpdump rsync open-vm-tools mlocate htop tar zip unzip  cloud-guest-utils
```

## Установка Nginx из репозитория

Скачиваем ключ подписи для репозитория Nginx

```sh
$ sudo wget https://nginx.org/keys/nginx_signing.key
```

Устанавливаем утилиту gnupg2

```sh
$ sudo apt -y install gnupg2
```

Добавляем загруженный ключ в список программных ключей

```sh
$ sudo apt-key add nginx_signing.key
```

Добавляем репозиторий Nginx

```sh
$ sudo nano /etc/apt/sources.list
...
# NGINX repo
deb https://nginx.org/packages/mainline/debian/ bullseye nginx
deb-src https://nginx.org/packages/mainline/debian bullseye nginx
```

Устанавливаем Nginx

```sh
$ sudo apt update
$ sudo apt -y install nginx
```

Запускаем web-сервер и добавляем его в автозагрузку

```sh
$ sudo systemctl start nginx
$ sudo systemctl enable nginx
```

Добавим пользователя nginx в группу www-data

```sh
$ sudo usermod -aG www-data nginx
```

> Если не добавить пользователя nginx в группу www-data, то в дальнейшем у пользователя nginx не будет хватать прав доступа к файлу /run/php/php7.4-fpm.sock, из-за чего не запустится установка DukuWiki (в логах Nginx будет появляться ошибка)

## Установка и настройка PHP-FPM 7.4

Устанавливаем php-fpm 7.4 и модули php

```sh
$ sudo apt install php7.4 php7.4-cli php7.4-fpm php7.4-curl php7.4-opcache php7.4-gd php7.4-xml php7.4-zip php7.4-json php7.4-mbstring php7.4-intl php7.4-imagick
```

Немного отредактируем конфиг php

```sh
$ sudo nano /etc/php/7.4/fpm/php.ini
...
short_open_tag = On

...
date.timezone = Europe/Moscow
```

Также отредактируем конфиг пула www для php-fpm

```sh
$ sudo nano /etc/php/7.4/fpm/pool.d/www.conf
...
user = www-data
group = www-data
listen.mode = 0660
```

На этом установка и настройка php-fpm завершена

## Подготовка к установке DokuWiki

Скачиваем релиз DokuWiki

```sh
$ cd
$ wget https://download.dokuwiki.org/src/dokuwiki/dokuwiki-stable.tgz
```

Создаем каталог, куда будет установлен DokuWiki

```sh
$ sudo mkdir -p /var/www/dokuwiki
```

Распаковываем архив с дистрибутивом в созданный каталог

```sh
$ sudo tar xzf dokuwiki-stable.tgz -C /var/www/dokuwiki/ --strip-components=1
```

Назначаем владельца директорий и файлов

```sh
$ sudo chown -R www-data:www-data /var/www/dokuwiki/
```

## Настройка Nginx

Отключаем дефолтный конфиг

```sh
$ sudo mv /etc/nginx/conf.d/default.conf /etc/nginx/conf.d/default.conf.disabled
```

Создаем конфиг для DokuWiki

```sh
$ sudo nano /etc/nginx/conf.d/dokuwiki.conf
server {

    listen [::]:443 ssl;
    listen 443 ssl;
    listen [::]:80;
    listen 80;
    ssl_certificate /etc/nginx/ssl/server.crt;
    ssl_certificate_key /etc/nginx/ssl/private.key;
    server_name _;
    root /var/www/dokuwiki;
    index index.html index.htm index.php doku.php;

    client_max_body_size 15M;
    client_body_buffer_size 128K;

    location / {
	    index doku.php;
        try_files $uri $uri/ @dokuwiki;
    }

    location ~ ^/lib.*\.(gif|png|ico|jpg)$ {
      expires 30d;
    }

    location ^~ /conf/ { return 403; }
    location ^~ /data/ { return 403; }
    location ~ /.ht { deny all; }

    location @dokuwiki {
        rewrite ^/_media/(.*) /lib/exe/fetch.php?media=$1 last;
        rewrite ^/_detail/(.*) /lib/exe/detail.php?media=$1 last;
        rewrite ^/_export/([^/]+)/(.*) /doku.php?do=export_$1&id=$2 last;
        rewrite ^/(.*) /doku.php?id=$1 last;
    }
    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass unix:/run/php/php7.4-fpm.sock;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param  QUERY_STRING     $query_string;
        fastcgi_param  REQUEST_METHOD   $request_method;
        fastcgi_param  CONTENT_TYPE     $content_type;
        fastcgi_param  CONTENT_LENGTH   $content_length;
        fastcgi_intercept_errors        on;
        fastcgi_ignore_client_abort     off;
        fastcgi_connect_timeout 60;
        fastcgi_send_timeout 180;
        fastcgi_read_timeout 180;
        fastcgi_buffer_size 128k;
        fastcgi_buffers 4 256k;
        fastcgi_busy_buffers_size 256k;
        fastcgi_temp_file_write_size 256k;
    }
}
```

Проверяем конфиг Nginx на ошибки

```sh
$ sudo nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

> ssl-сертификат у меня уже был

Перезапускаем службы

```sh
$ sudo systemctl restart php7.4-fpm nginx
```

Устанавливаем и настраиваем DokuWiki. Для это переходим на наш сайт

```
https://%mysite%/install.php?l=ru
```

После установки DokuWiki удаляем файл install.php

```sh
$ sudo rm -f /var/www/dokuwiki/install.php
```

На этом установка DokuWiki в Debian 11 завершена
