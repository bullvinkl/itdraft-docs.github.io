---
title: "Установка корпоративного менеджера паролей Passwork + Nginx в Centos 8 / Rocky Linux"
date: "2021-10-12"
categories: 
  - Security-System
tags: 
  - "centos"
  - "nginx"
  - "passwork"
  - "php-fpm"
  - "rocky-linux"
image:
  path: /commons/evillimiter-featured.png
  alt: "Установка корпоративного менеджера паролей Passwork"
---

> **Passwork** хранит пароли в структурированном виде с гибкой настройкой доступов пользователей и подходит как для совместной работы внутри компаний, так и для личного использования.
{: .prompt-tip }

## Подготовка

Устанавливаем софт

```sh
$ sudo dnf makecache
$ sudo dnf -y install epel-release
$ sudo dnf -y install wget traceroute net-tools nano bind-utils telnet htop rsync policycoreutils-python-utils
$ sudo dnf-y install git avahi
```

## Устанавливаем Nginx, настраиваем Firewall

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

Устанавливаем Nginx, добавляем его в автозагрузку и запускаем

```sh
$ sudo dnf -y install nginx
$ sudo systemctl enable --now nginx
```

Настраиваем Firewall

```sh
$ sudo firewall-cmd --permanent --add-service=http
$ sudo firewall-cmd --permanent --add-service=https
$ sudo firewall-cmd --permanent --add-port=5353/udp
$ sudo firewall-cmd --reload
```

Перезапускаем сервис `avahi`

```sh
$ sudo systemctl restart avahi-daemon
```

## Установка базы данных MongoDB 4.2

Добавляем репозиторий MongoDB

```sh
$ sudo nano /etc/yum.repos.d/mongodb-org-4.2.repo

[mongodb-org-4.2]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/4.2/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-4.2.asc
```

Устанавливаем MongoDB

```sh
$ sudo dnf -y install mongodb-org
```

Отключаем SELinux

```sh
$ sudo nano /etc/selinux/config
...
SELINUX=disabled
```

Что б не перезагружаться выполняем команду

```sh
$ sudo setenforce 0
```

Добавляем MongoDB в автозагрузку, запускаем, проверяем

```sh
$ sudo systemctl enable --now mongod
$ systemctl status mongod
```

## Установка PHP-fpm 7.3

Добавляем репозиторий Remi

```sh
$ sudo dnf -y install https://rpms.remirepo.net/enterprise/remi-release-8.rpm
```

Устанавливаем PHP-FPM 7.3 и необходимые модули

```sh
$ sudo dnf module list php
$ sudo dnf module enable php:remi-7.3
$ sudo dnf -y install php-fpm php-json php-ldap php-xml php-bcmath php-mbstring
```

Настраиваем PHP-FPM

```sh
$ sudo nano /etc/php-fpm.d/www.conf
...
user = nginx
group = nginx
...
listen = /run/php-fpm/www.sock
```

Редактируем `php.ini`

```sh
$ sudo nano /etc/php.ini
...
date.timezone = Europe/Moscow
...
short_open_tag = On
```

Меняем владельца директории

```sh
$ sudo chown -R nginx. /var/lib/php/session
```

Добавляем PHP-FPM в автозагрузку, запускаем, проверяем

```sh
$ sudo systemctl enable --now php-fpm
$ systemctl status php-fpm
```

## Установка драйвера PHP Mongo

Устанавливаем необходимые компоненты

```sh
$ sudo dnf -y install gcc php-pear openssl-devel
```

В Centos 8 для установки `php-devel` необходимо вначале установить пакет `libedit-devel` из репозитория `PowerTools`, иначе в консоли будет ошибка установки

```sh
$ sudo dnf -y install https://mirror.yandex.ru/centos/8/PowerTools/x86_64/os/Packages/libedit-devel-3.1-23.20170329cvs.el8.x86_64.rpm
$ sudo dnf -y install php-devel
```

Собираем драйвер

```sh
$ sudo pecl install mongodb
```

Добавляем его в PHP

```sh
$ echo "extension=mongodb.so" | sudo tee /etc/php.d/20-mongodb.ini
```

Перезапускаем сервис PHP-FPM

```sh
$ sudo systemctl restart php-fpm
```

## Установка PHP фреймворка Phalcon версии 3.4.5

Устанавливаем необходимые компоненты

```sh
$ sudo yum -y install php-mysql libtool pcre-devel
```

Клонируем репозиторий и устанавливаем фреймворк

```sh
$ cd /opt/
$ sudo git clone --branch 3.4.x  --depth=1 "https://github.com/phalcon/cphalcon.git"
$ cd cphalcon/build
$ sudo ./install
```

Добавляем его в PHP

```sh
$ echo "extension=phalcon.so" | sudo tee /etc/php.d/50-phalcon.ini
```

Перезапускаем сервис PHP-FPM

```sh
$ sudo systemctl restart php-fpm
```

Проверяем, подгрузились ли в PHP модули

```sh
$ php -m | egrep 'phalcon|mongodb'
```

## Загрузка и установка Passwork

Создаем каталог и переходим в него

```sh
$ sudo mkdir /opt/passwork
$ cd /opt/passwork
```

Клонируем репозиторий

```sh
$ sudo git init
$ sudo git remote add origin https://passwork.download/passwork/passwork.git
$ sudo git fetch
login:
pass:

$ sudo git checkout v4
```

> Логин, пароль и сертификат можно запросить в [https://passwork.ru/](https://passwork.ru/) для тестирования

Копируем конфигурационный файл и меняем права на файлы / каталоги

```sh
$ sudo cp /opt/passwork/app/config/config.example.ini /opt/passwork/app/config/config.ini
$ sudo find /opt/passwork/ -type d -exec chmod 755 {} \;
$ sudo find /opt/passwork/ -type f -exec chmod 644 {} \;
$ sudo chown -R nginx. /opt/passwork/
```

Редактируем конфигурационный файл

```sh
$ sudo nano /opt/passwork/app/config/config.ini
...
[application]
domain = http://passwork.itdraft.ru
```

Восстанавливаем базу данных MongoDB

```sh
$ sudo mongorestore --drop /opt/passwork/dump/
```

Настраиваем Nginx. Отключаем дефолтный конфиг nginx

```sh
$ sudo mv /etc/nginx/conf.d/default.conf /etc/nginx/conf.d/default.conf_disabled
```

Копируем конфиг репозитория

```sh
$ sudo cp /opt/passwork/nginx.conf.example /etc/nginx/conf.d/nginx.conf
```

Настраиваем

```sh
$ sudo nano /etc/nginx/conf.d/nginx.conf
    server unix:/run/php-fpm/www.sock;

    root /opt/passwork/public/;
    server_name passwork.itdraft.ru;
```

Перезапускаем сервисы

```sh
$ sudo systemctl restart nginx php-fpm
```

## Подключаем SSL-сертификат, для работы расширения браузера

Генерируем самоподписанный сертификат, или используем готовый

```sh
$ sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -subj '/CN=passwork.itdraft.ru' -addext 'subjectAltName=DNS: passwork.itdraft.ru' -keyout /etc/ssl/certs/passwork.key -out /etc/ssl/certs/passwork.crt
```

Создаем группу Диффи-Хеллмана

```sh
$ sudo openssl dhparam -out /etc/ssl/certs/dhparam.pem 2048
```

Добавляем ее в наш сгенерированный сертификат

```sh
$ cat /etc/ssl/certs/dhparam.pem | sudo tee -a /etc/ssl/certs/passwork.crt
```

При работе через SSL-соединение (HTTPS) браузер Chrome требует наличия флагов `Secure` и `SameSite` в cookie. Включаем их

```sh
$ sudo nano /etc/php.ini
...
session.cookie_secure = On
```

Выключаем параметр `disableSameSiteCookie` в конфигурационном файле Passwork `config.ini`

```sh
$ sudo nano /opt/passwork/app/config/config.ini
[application]

...
disableSameSiteCookie = Off
```

Перезагружаем сервисы PHP-FPM и Nginx

```sh
$ sudo systemctl restart php-fpm nginx
```

Донастраиваем конфиг Nginx

```sh
$ sudo nano /etc/nginx/conf.d/nginx.conf

server {
    listen 80;
    server_name passwork.itdraft.ru;
    rewrite ^ https://$http_host$request_uri? permanent;        # force redirect http to https
    server_tokens off;
}

server {
    listen 443 ssl;
    server_name pass.gge.local;
#    ssl on;
    ssl_certificate /etc/ssl/certs/passwork.crt;        # path to your cacert.pem
    ssl_certificate_key /etc/ssl/certs/passwork.key;      # path to your privkey.pem
    ssl_session_timeout 5m;
    ssl_session_cache shared:SSL:5m;

    # Diffie-Hellman parameter for DHE ciphersuites, recommended 2048 bits
    ssl_dhparam /etc/ssl/certs/dhparam.pem;

    # secure settings (A+ at SSL Labs ssltest at time of writing)
    # see https://wiki.mozilla.org/Security/Server_Side_TLS#Nginx
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers 'ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES256-SHA384:ECDHE-RSA-AES128-SHA256:ECDHE-RSA-AES256-SHA:ECDHE-ECDSA-AES256-SHA:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES128-SHA:DHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES256-SHA256:DHE-RSA-AES256-SHA:DHE-RSA-CAMELLIA256-SHA:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-RSA-SEED-SHA:DHE-RSA-CAMELLIA128-SHA:HIGH:!aNULL:!eNULL:!LOW:!3DES:!MD5:!EXP:!PSK:!SRP:!DSS';
    ssl_prefer_server_ciphers on;
...
```

Готово, теперь можно подключать расширение для браузера
