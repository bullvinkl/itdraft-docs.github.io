---
title: "Установка и подключение модуля компрессии Brotli для NGINX в Centos 7"
date: "2019-09-06"
categories: 
  - Linux
  - Nginx
tags: 
  - "brotli"
  - "centos"
  - "nginx"
image:
  path: /commons/1051548_1085_2.jpg
  alt: "Подключение модуля компрессии Brotli"
---

> **Brotli** - это новый алгоритм сжатия, который теперь широко поддерживается во многих браузерах. Метод сжатия brotli основан на современном варианте алгоритма LZ77. 
> По сравнению с классическим алгоритмом deflate (середина 1990-х, ZIP, gzip), brotli, как правило, достигает на 20% более высокую степень сжатия для текстовых файлов, сохраняя сходную скорость сжатия и распаковки.
{: .prompt-tip }

Добавим репозиторий GetPageSpeed

```sh
$ sudo yum -y install https://extras.getpagespeed.com/release-el7-latest.rpm
```

Ранее была рассмотрена [установка Nginx]({% post_url 2019-07-15-ustanovka-web-servera-nginx-dlya-raboty-s-virtualnymi-hostami-php-fpm-v-rezhime-raboty-sock-mysql-server-mariadb-na-centos-7 %})

Установим модуль `nginx-module-nbr`

```sh
$ sudo yum -y install nginx-module-nbr
```

Откроем основной конфиг Nginx и подключим модуль

```sh
$ sudo nano /etc/nginx/nginx.conf
...
# Load dynamic modules. See /usr/share/nginx/README.dynamic.
include /usr/share/nginx/modules/*.conf;
load_module "modules/ngx_http_brotli_filter_module.so";
load_module "modules/ngx_http_brotli_static_module.so";
...
```

Подключим Brotli компрессию во всех наших сайтах, для этого создадим файл с соответствующим содержимым

```sh
$ sudo nano /etc/nginx/conf.d/brotli.conf
brotli on;
brotli_types text/xml
       image/svg+xml
       application/x-font-ttf
       image/vnd.microsoft.icon
       application/x-font-opentype
       application/json
       font/eot
       application/vnd.ms-fontobject
       application/javascript
       font/otf
       application/xml
       application/xhtml+xml
       text/javascript
       application/x-javascript
       text/plain
       application/x-font-truetype
       application/xml+rss
       image/x-icon
       font/opentype
       text/css
       image/x-win-bitmap;
brotli_comp_level 4;
```

Проверяем конфигурацию Nginx на ошибки и перезагружаем его

```sh
$ sudo nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful

$ sudo systemctl restart nginx
```

Проверяем, подключилось ли сжатие Brotli на сайте

```sh
$ sudo curl -IL https://itdraft.ru -H "Accept-Encoding: br"
...
Content-Encoding: br
```
