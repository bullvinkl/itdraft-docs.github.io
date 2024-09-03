---
title: "Включаем компрессию Brotli для Nginx в CentOS 8"
date: "2020-07-28"
categories: 
  - Linux
  - Nginx
tags: 
  - "brotli"
  - "centos"
  - "nginx"
image:
  path: /commons/1400240_c61f_3.jpg
  alt: "Brotli для Nginx"
---

Ранее была рассмотрена статья по установки модуля компрессии [Brotli для Nginx в Centos 7]({% post_url 2019-09-06-ustanovka-i-podkljuchenie-modulya-kompressii-brotli-dlya-nginx-v-centos-7 %}), но т.к. репозиторий, откуда устанавливался модуль перешел на платную основу, расмотрим установку модуля компрессии Brotli из исходников

## Подготовка

Обновляемся

```sh
$ sudo dnf update -y
```

Подключаем репозиторий EPEL

```sh
$ sudo dnf -y install epel-release
```

Устанавливаем необходимые пакеты

```sh
$ sudo dnf -y install nano curl wget git unzip socat bash-completion socat yum-utils
```

Устанавливаем Development Tools

```sh
$ sudo dnf -y groupinstall "Development Tools"
```

## Установка Nginx

Добавляем репозиторий Nginx

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

По-умолчанию будет использоваться ветка `nginx-stable`, если надо переключиться на `nginx-mainline`, выполняем команду

```sh
$ sudo yum-config-manager --enable nginx-mainline
```

Устанавливаем Nginx

```sh
$ sudo dnf -y install nginx
```

Смотрим версию

```sh
$ nginx -v
nginx version: nginx/1.18.0
```

Запускаем Nginx и добавляем его в автозагрузку

```sh
$ sudo systemctl enable --now nginx
```

## Установка модуля Brotli из исходников

После установки Nginx нам нужно собрать модуль Brotli `ngx_brotli` как динамический модуль Nginx. Начиная с версии 1.11.5 Nginx, можно скомпилировать отдельные динамические модули без компиляции полного программного обеспечения Nginx

Скачиваем исходники nginx и распаковываем

```
$ wget https://nginx.org/download/nginx-1.18.0.tar.gz
$ tar zxvf nginx-1.18.0.tar.gz
```

> Важно, чтобы номера версий Nginx и версия исходников Nginx совпадали. Если вы установили Nginx из репозитория, то вы должны загрузить ту же версию исходников
{: .prompt-warning }

Удаляем архив `nginx-1.18.0.tar.gz`

```sh
$ rm nginx-1.18.0.tar.gz
```

Клонируем `ngx_brotli` из GitHub репозитория

```sh
$ git clone https://github.com/google/ngx_brotli.git
$ cd ngx_brotli
$ git submodule update --init 
$ cd ~
```

Переходим в каталог с исходниками Nginx

```sh
$ cd ~/nginx-1.18.0
```

Устанавливаем недостающие библиотеки

```sh
$ sudo dnf -y install pcre pcre-devel zlib zlib-devel openssl openssl-devel
```

Компилируем модуль `ngx_brotli` и копируем результат в директорию Nginx

```sh
$ ./configure --with-compat --add-dynamic-module=../ngx_brotli
$ make modules
$ sudo cp objs/*.so /etc/nginx/modules
```

Проверяем

```sh
$ ls /etc/nginx/modules
ngx_http_brotli_filter_module.so  ngx_http_brotli_static_module.so
```

Выставляем права на файлы

```sh
$ sudo chmod 644 /etc/nginx/modules/*.so
```

## Настройка Nginx

Редактируем конфиг `nginx.conf`, подключаем модули

```sh
$ sudo nano /etc/nginx/nginx.conf
[...]
# Load Brotli
load_module modules/ngx_http_brotli_filter_module.so;
load_module modules/ngx_http_brotli_static_module.so;

events {
[...]
```

Проверяем

```sh
$ sudo nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

Редактируем дефолтный конфиг web-сервера, включаем компрессию

```sh
$ sudo nano /etc/nginx/conf.d/default.conf
[...]
  brotli on;
  brotli_static on;
  brotli_types text/plain text/css text/javascript application/javascript text/xml application/xml image/svg+xml application/json;
  brotli_comp_level 4;
}
```

Проверяем

```sh
$ sudo nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

Перезагружаем Nginx

```sh
$ sudo systemctl reload nginx
```

## Настройка Firewall

Открываем 80 порт

```sh
$ sudo firewall-cmd --zone=public --add-service=http --permanent
$ sudo firewall-cmd --reload
```

Открываем в браузере ip-адрес сервера

![](/assets/img/posts/2020/07/28/nginx_br.png){: w="300" }
_Проверка_