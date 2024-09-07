---
title: "Включаем SSL в Nginx на Centos 7"
date: "2019-07-27"
categories: 
  - Web
tags: 
  - "centos"
  - "nginx"
  - "ssl"
image:
  path: /commons/910838_84d6.jpg
  alt: "Включаем SSL в Nginx"
---

> **SSL** — криптографический протокол, который подразумевает более безопасную связь. Он использует асимметричную криптографию для аутентификации ключей обмена, симметричное шифрование для сохранения конфиденциальности, коды аутентификации сообщений для целостности сообщений.
{: .prompt-tip }

[Установка Nginx]({% post_url 2019-07-15-ustanovka-web-servera-nginx-dlya-raboty-s-virtualnymi-hostami-php-fpm-v-rezhime-raboty-sock-mysql-server-mariadb-na-centos-7 %}) была рассмотрена ранее

Генерируем 2048-битный файл с шифрами для DH. Этот файл позволяет генерировать «одноразовые» ключи, которые используются при обмене сессионными ключами. Они уничтожаются по окончанию ssl-хендшейка, что затрудняет последующее восстановление сессионного ключа из дампа трафика (перехватить нужно будет уже не только приватный ключ и дамп трафика, но и файл с dhparam — без всего этого восстановить содержимое tls-трафика уже не получится).

```sh
$ sudo openssl dhparam -out /etc/ssl/certs/dhparam.pem 2048
```

Создадим каталог, в котором будет лежать файл с настройками для ssl. Это нужно, что б избежать дублирования в конфигурациях для виртуальных хостов

```sh
$ sudo mkdir /etc/nginx/snippets
$ sudo nano /etc/nginx/snippets/ssl-params.conf
ssl_dhparam /etc/ssl/certs/dhparam.pem;

ssl_session_timeout 1d;
ssl_session_cache shared:SSL:50m;
ssl_session_tickets off;

ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
ssl_ciphers 'ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA:ECDHE-RSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-RSA-AES256-SHA256:DHE-RSA-AES256-SHA:ECDHE-ECDSA-DES-CBC3-SHA:ECDHE-RSA-DES-CBC3-SHA:EDH-RSA-DES-CBC3-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:DES-CBC3-SHA:!DSS';
ssl_prefer_server_ciphers on;

ssl_stapling on;
ssl_stapling_verify on;
resolver 8.8.8.8 8.8.4.4 valid=300s;
resolver_timeout 30s;

add_header Strict-Transport-Security "max-age=15768000; includeSubdomains; preload";
add_header X-Frame-Options SAMEORIGIN;
add_header X-Content-Type-Options nosniff;
```

После покупки сертификата нам понадобятся следующие файлы:

```
.CA - файл сертификата Центра Сертификации (Certificate Authority).
.CRT - файл сертификата вашего веб-сайта.
.key - закрытый ключ. Он мог быт сгенерирован на вашем сервере, либо при покупке ssl сертификата на web-сайте: private.key
```

Объединяем .ca и .crt в один файл

```sh
$ cat /etc/ssl/certs/mydomain.ru_crt.crt /etc/ssl/certs/mydomain.ru_ca.crt | sudo tee -a /etc/ssl/certs/mydomain.ru.crt
```

Теперь нам надо добавить в наш конфиг nginx строчки:

```
server {
        listen 443 ssl default_server;
	include snippets/ssl-params.conf;
        root /var/www/html;

        server_name mydomain.ru www.mydomain.ru;
	index index.php index.html index.htm;

        ssl_certificate /etc/ssl/mydomain.ru.crt;
        ssl_certificate_key /etc/ssl/private.key;
		...
```

Вот готовый пример конфига

```sh
$ cd /etc/nginx/sites-available/
$ cat mydomain.ru.conf
# Переадресация HTTP -> HTTPS
server {
    listen 80;
    server_name mydomain.ru www.mydomain.ru;
    return 301 https://$server_name$request_uri;
}

# Переадресация IP -> Домен
server {
    listen 80;
    server_name 8.8.8.8;
    return 301 https://mydomain.ru$request_uri;
}

# Переадресация WWW -> NON WWW
server {
    listen 443 ssl http2;
    server_name www.mydomain.ru;
    return 301 https://mydomain.ru$request_uri;

    include snippets/ssl-params.conf;
    ssl_certificate /etc/ssl/certs/mydomain.ru.crt;
    ssl_certificate_key /etc/ssl/certs/mydomain.ru_key.key;
}

server {
    listen 443 ssl http2;
    server_name mydomain.ru;

    root $root_path;
    set $root_path /var/www/mydomain.ru/public_html;
    set $php_sock unix:/var/run/php-fpm/php-fpm.sock;
    index index.php index.html;

    client_max_body_size 1024M;
    client_body_buffer_size 4M;

    # включаем сжатие gzip
    gzip on;
    gzip_disable "msie6";
    gzip_vary on;
    gzip_proxied any;
    gzip_comp_level 6;
    gzip_buffers 16 8k;
    gzip_types text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript application/javascript;

    # SSL parameters
    include snippets/ssl-params.conf;
    ssl_certificate /etc/ssl/certs/mydomain.ru.crt;
    ssl_certificate_key /etc/ssl/certs/mydomain.ru_key.key;
    
    # log files
    access_log /var/www/mydomain.ru/logs/access.log;
    error_log /var/www/mydomain.ru/logs/error.log;

    location = /favicon.ico {
        log_not_found off;
        access_log off;
    }

    location = /robots.txt {
        allow all;
        log_not_found off;
        access_log off;
    }

    # запрет для загруженных скриптов
    location ~* /(?:uploads|files)/.*\.php$ {
        deny all;
    }

    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    location ~ \.php$ {
        include fastcgi_params;
        fastcgi_pass $php_sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        try_files $uri =404;
    }

    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg)$ {
        expires max;
        log_not_found off;
    }

    # ht(passwd|access)
    location ~* /\.ht {
        deny all;
    }

    location ~ /\. {
        deny all;
        access_log off;
        log_not_found off;
    }

}
```
