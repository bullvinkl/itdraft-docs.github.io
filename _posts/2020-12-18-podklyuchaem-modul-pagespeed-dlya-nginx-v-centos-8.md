---
title: "Подключаем модуль PageSpeed для Nginx в Centos 8"
date: "2020-12-18"
categories: 
  - Linux
  - Nginx
tags: 
  - "nginx"
  - "pagespeed"
image:
  path: /commons/featured.jpg
  alt: "Подключаем модуль PageSpeed для Nginx"
---

> **Pagespeed** (или **ngx_pagespeed**) – это модуль для web-сервера Nginx и Apache с открытым исходным кодом, используемый для повышения скорости работы сайтов путём сокращения времени загрузки сайта в браузере.

Устанавливаем необходимый софт для того, чтобы собрать модуль из исходников

```sh
$ sudo dnf -y install wget curl unzip gcc-c++ pcre-devel zlib-devel
$ sudo dnf -y install gcc-c++ pcre-devel zlib-devel make unzip libuuid-devel
```

Создаем директорию, куда будем закачивать архивы с исходным кодом. В последствии ее можно будет удалить

```sh
$ mkdir ~/nginx
$ cd ~/nginx
```

Смотрим версию Nginx

```sh
$ nginx -v
nginx version: nginx/1.18.0
```

Скачиваем и разархивируем эту же версию Nginx

```sh
$ wget http://nginx.org/download/nginx-1.18.0.tar.gz
$ tar -xvzf nginx-1.18.0.tar.gz
```

Скачиваем и разархивируем исходники PageSpeed

```sh
$ wget https://github.com/apache/incubator-pagespeed-ngx/archive/v1.13.35.2-stable.tar.gz
$ tar -xvzf v1.13.35.2-stable.tar.gz
```

Скачиваем ПО `PSOL` и распаковываем в каталог с модулем `pagespeed`

```sh
$ cd incubator-pagespeed-ngx-1.13.35.2-stable
$ wget https://dl.google.com/dl/page-speed/psol/1.13.35.2-x64.tar.gz
$ tar -xvzf 1.13.35.2-x64.tar.gz
```

Запускаем процесс конфигурирования Nginx

```sh
$ cd ~/nginx/nginx-1.18.0/
$ sudo ./configure --with-compat --add-dynamic-module=../incubator-pagespeed-ngx-1.13.35.2-stable
[...]
Configuration summary
  + using system PCRE library
  + OpenSSL library is not used
  + using system zlib library

  nginx path prefix: "/usr/local/nginx"
  nginx binary file: "/usr/local/nginx/sbin/nginx"
  nginx modules path: "/usr/local/nginx/modules"
  nginx configuration prefix: "/usr/local/nginx/conf"
  nginx configuration file: "/usr/local/nginx/conf/nginx.conf"
  nginx pid file: "/usr/local/nginx/logs/nginx.pid"
  nginx error log file: "/usr/local/nginx/logs/error.log"
  nginx http access log file: "/usr/local/nginx/logs/access.log"
  nginx http client request body temporary files: "client_body_temp"
  nginx http proxy temporary files: "proxy_temp"
  nginx http fastcgi temporary files: "fastcgi_temp"
  nginx http uwsgi temporary files: "uwsgi_temp"
  nginx http scgi temporary files: "scgi_temp"
```

Начиная с версии Nginx 1.11.5 можно скомпилировать отдельные динамические модули без компиляции полного программного обеспечения Nginx.
Компилируем модуль `ngx_pagespeed`

```sh
$ sudo make modules
```

Копируем его в директорию nginx

```sh
$ sudo cp objs/ngx_pagespeed.so /etc/nginx/modules
$ sudo chmod 644 /etc/nginx/modules/ngx_pagespeed.so
```

Добавляем модуль `ngx_pagespeed.so` в конфиг Nginx

```sh
$ sudo nano /etc/nginx/nginx.conf
...
pid        /var/run/nginx.pid;

load_module modules/ngx_pagespeed.so;

events {
...
```

Подключим модуль PageSpeed во всех наших сайтах, для этого создадим файл с соответствующим содержимым

```sh
$ sudo nano /etc/nginx/conf.d/pagespeed.conf
pagespeed on;
pagespeed FileCachePath /var/cache/pagespeed;
pagespeed HttpCacheCompressionLevel 0;

# HTTPS Support
pagespeed FetchHttps enable;

# PageSpeed Filters
# CSS Minification
pagespeed EnableFilters combine_css,rewrite_css;

# JS Minification
pagespeed EnableFilters combine_javascript,rewrite_javascript;

# Images Optimization
pagespeed EnableFilters lazyload_images;
pagespeed EnableFilters rewrite_images;
pagespeed EnableFilters convert_jpeg_to_progressive,convert_png_to_jpeg,convert_jpeg_to_webp,convert_to_webp_lossless;

# Remove comments from HTML
pagespeed EnableFilters remove_comments;

# Remove WHITESPACE from HTML
pagespeed EnableFilters collapse_whitespace;
```

Проверяем

```sh
$ sudo nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

Перезапускаем nginx и смотрим статус

```sh
$ sudo systemctl restart nginx
$ systemctl restart nginx
```

Смотрим, отрабатывает ли pagespeed

```sh
$ curl -I localhost
HTTP/1.1 200 OK
Server: nginx/1.18.0
Content-Type: text/html
Connection: keep-alive
Vary: Accept-Encoding
Date: Fri, 18 Dec 2020 13:46:53 GMT
X-Page-Speed: 1.13.35.2-0
Cache-Control: max-age=0, no-cache
```