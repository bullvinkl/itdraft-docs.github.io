---
title: "Установка NGINX и PHP-FPM в Debian 10"
date: "2019-08-27"
categories: 
  - Linux
  - Nginx
tags: 
  - "debian"
  - "nginx"
  - "php-fpm"
image:
  path: /commons/1334578_287d_2.jpg
  alt: "Установка NGINX и PHP-FPM в Debian"
---

> **PHP-FPM** — это разновидность Server API (SAPI) для PHP, отвечающая за управление процессами PHP с целью ускорения и Improve stability при использовании в сценариях, требующих высокой производительности и масштабируемости.
> PHP-FPM отличается от традиционного использования PHP с Apache тем, что он работает как standalone процесс, не требующий веб-сервера для запуска. Вместо этого, он взаимодействует Directly с web-сервером Nginx или IIS для обработки запросов.
{: .prompt-tip }

Обновляемся

```sh
$ sudo apt update
```

## Устанавливаем NGINX

```sh
$ sudo apt install nginx
```

Если у вас не установлен файерволл UFW, то установим его

```sh
$ sudo apt install ufw
```

Открываем 80 порт в файерволле и перезагружаем

```sh
$ sudo ufw allow 'Nginx HTTP'
$ sudo ufw reload
```

Проверяем статус

```sh
$ sudo ufw status
Status: active
To                         Action      From
-- ------ ----
OpenSSH                    ALLOW       Anywhere
Nginx HTTP                 ALLOW       Anywhere
OpenSSH (v6)               ALLOW       Anywhere (v6)
Nginx HTTP (v6)            ALLOW       Anywhere (v6)
```

Чтобы открыть и 80 и 443 порт, т.е. http и https надо выполнить команду

```sh
$ sudo ufw allow 'Nginx FULL'
```

## Устанавливаем PHP-FPM

Поскольку Nginx не содержит нативную обработку PHP, нам нужно установить fpm, что означает «менеджер процессов fastCGI». Мы скажем Nginx передать PHP-запросы этому программному обеспечению для обработки

```sh
$ apt install php-fpm
```

Если требуются дополнительные модули, установим их

```sh
$ apt install php-mysql php-bcmath php-ctype php-json php-mbstring php-pdo php-tokenizer php-xml php-curl
```

## Настраиваем Nginx

Создадим каталоги для сайта и логов

```sh
$ sudo mkdir -p /var/www/%site_name%/{htdocs,logs}
```

Теперь создадим файл с конфигурацией для виртуального хоста в Nginx

```sh
$ sudo nano /etc/nginx/sites-available/%site_name%.conf
server {
    listen 80;
    listen [::]:80;

    root $root_path;
    set $root_path /var/www/%site_name%/htdocs;
    set $php_sock unix:/var/run/php/php7.3-fpm.sock;
    index index.php index.html index.htm;

    server_name %site_name%;

    access_log /var/www/%site_name%/logs/access.log;
    error_log /var/www/%site_name%/logs/error.log;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass $php_sock;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }
}
```

Делаем сим линк, что бы подключить виртуальный хост

```sh
$ sudo ln -s /etc/nginx/sites-available/%site_name%.conf /etc/nginx/sites-enabled/
```

Проверяем конфигурацию Nginx, что бы убедиться, что в ней нет ошибок

```sh
$ nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

Перезапускаем Nginx и PHP-FPM

```sh
$ systemctl restart nginx php7.3-fpm
```

Добавляем имя нашего сайта в файл `/etc/hosts`

```sh
$ sudo nano /etc/hosts
127.0.0.1    %site_name%
```
