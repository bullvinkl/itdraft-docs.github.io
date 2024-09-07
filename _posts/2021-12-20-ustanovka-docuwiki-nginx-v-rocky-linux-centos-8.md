---
title: "Установка DokuWiki + Nginx в Rocky Linux / Centos 8"
date: "2021-12-20"
categories: 
  - Content-Management
tags: 
  - centos
  - dokuwiki
  - php
  - php-fpm
  - rocky-linux
  - firewall
  - selinux
image:
  path: /commons/laptops.png
  alt: "Установка DokuWiki"
---

> **DokuWiki** — простой, но достаточно мощный вики-движок, который может быть использован для создания любой документации. Автор проекта — Андреас Гор. В отличие от многих других движков, DokuWiki использует для хранения страниц текстовые файлы, таким образом единственным требованием является поддержка хостингом PHP.
{: .prompt-tip }

## Подготовка

Устанавливаем утилиты

```sh
$ sudo dnf -y install curl wget vim nano git unzip socat bash-completion
```

Подключаем репозиторий EPEL

```sh
$ sudo dnf -y install epel-release
```

## Установка Nginx

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

Устанавливаем Nginx

```sh
$ sudo dnf -y update
$ sudo dnf -y install nginx
```

Запускаем nginx и добавляем его в автозагрузку

```sh
$ sudo systemctl enable --now nginx
```

Отключаем дефолтный конфиг

```sh
$ sudo mv /etc/nginx/conf.d/default.conf /etc/nginx/conf.d/default.conf.disable
```

## Настройка Firewall

Открываем порты 80 и 443

```sh
$ sudo firewall-cmd --permanent --add-service=http
$ sudo firewall-cmd --permanent --add-service=https
$ sudo firewall-cmd --reload
```

## Настройка SELinux

Устанавливаем утилиту для управления правилами SELinux

```sh
$ sudo dnf -y install policycoreutils-python-utils
```

Добавляем правило

```sh
$ sudo semanage permissive -a httpd_t
```

## Установка PHP

Устанавливаем необходимые компоненты

```sh
$ sudo dnf -y install php php-cli php-fpm php-gd php-xml php-zip php-json
```

Настраиваем php-fpm

```sh
$ sudo nano /etc/php-fpm.d/www.conf
...
user = nginx
...
group = nginx
...
```

Меняем владельца каталога

```sh
$ sudo chown -R root:nginx /var/lib/php/session
```

Добавляем сервис в автозагрузку и запускаем его

```sh
$ sudo systemctl start php-fpm
$ sudo systemctl enable php-fpm
```

## Настройка Nginx

Создадим директорию для ssl-сертификата

```sh
$ sudo mkdir /etc/nginx/ssl/
```

Сгенерируем самоподписанный сертификат и ключ

```sh
$ sudo openssl req -x509 -sha256 -nodes -days 365 -newkey rsa:4096 -keyout /etc/nginx/ssl/private.key -out /etc/nginx/ssl/server.crt
```

Можно так же использовать покупной сертификат, либо бесплатный lets encrypt, но т.к. я разворачиваю DokuWiki в тестовой среде, использую самоподписанный

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
        try_files $uri $uri/ @dokuwiki;
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
    location ~ .php$ {
        try_files $uri =404;
        fastcgi_pass unix:/run/php-fpm/www.sock;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }
}
```

Перезапускаем службы

```sh
$ sudo systemctl restart php-fpm nginx
```

## Установка DokuWiki

Скачиваем дистрибутив

```sh
$ cd /tmp
$ wget https://download.dokuwiki.org/src/dokuwiki/dokuwiki-stable.tgz
```

Создаем каталог, где будет лежать дистрибутив

```sh
$ sudo mkdir -p /var/www/dokuwiki
```

Распаковываем DokuWiki и переносим файлы в созданный каталог

```sh
$ tar xvf dokuwiki-stable.tgz
$ cd dokuwiki-2020-07-29
$ sudo mv * /var/www/dokuwiki
```

Меняем владельца каталога и файлов

```sh
$ sudo chown -R nginx:nginx /var/www/dokuwiki
```

Далее переходим в web-интерфейс по ip-адресу нашего сервера и завершаем установку DokuWiki
