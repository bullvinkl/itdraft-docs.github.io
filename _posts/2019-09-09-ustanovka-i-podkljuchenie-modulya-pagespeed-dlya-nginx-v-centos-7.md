---
title: "Установка и подключение модуля PageSpeed для Nginx в Centos 7"
date: "2019-09-09"
categories: 
  - Linux
  - Nginx
tags: 
  - "centos"
  - "nginx"
  - "pagespeed"
image:
  path: /commons/DC79MkJWAAA1qod.jpg
  alt: "Подключение модуля PageSpeed"
---

> **PageSpeed** - это инструмент, разработанный Google, для оценки производительности и скорости загрузки веб-страниц. Он измеряет время, необходимое для отображения контента на странице, начиная от начала первой отрисовки контента (FCP) до момента полной инициализации интерфейса для работы пользователя (DCL). Чем меньше это время, тем быстрее загружается страница в браузере.
{: .prompt-tip }

Добавим репозиторий GetPageSpeed

```sh
$ sudo yum -y install https://extras.getpagespeed.com/release-el7-latest.rpm
```

[Установка Nginx]({% post_url 2019-07-15-ustanovka-web-servera-nginx-dlya-raboty-s-virtualnymi-hostami-php-fpm-v-rezhime-raboty-sock-mysql-server-mariadb-na-centos-7 %}) была рассмотрена раньше

Установим модуль PageSpeed

```sh
$ sudo yum -y install nginx-module-pagespeed
```

Откроем основной конфиг Nginx и подключим модуль

```sh
$ sudo nano /etc/nginx/nginx.conf
...
# Load dynamic modules. See /usr/share/nginx/README.dynamic.
include /usr/share/nginx/modules/*.conf;
load_module modules/ngx_pagespeed.so;
...
```

Подключим модуль PageSpeer во всех наших сайтах, для этого создадим файл с соответствующим содержимым

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

Немного о параметре:

```
pagespeed HttpCacheCompressionLevel 0;
```

В своем нынешнем виде модуль PageSpeed не поддерживает внутреннее сжатие Brotli. То есть вы все равно можете использовать его вместе с модулем Brotli Nginx, но вам придется отключить внутреннее сжатие PageSpeed через.

Проверяем конфигурацию Nginx на ошибки и перезагружаем его

```sh
$ sudo nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
$ sudo systemctl restart nginx
```
