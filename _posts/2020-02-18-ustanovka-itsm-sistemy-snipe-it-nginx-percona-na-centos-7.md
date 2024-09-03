---
title: "Установка ITSM-системы Snipe-IT + NGINX + Percona в Centos 7"
date: "2020-02-18"
categories: 
  - Linux
  - ITSN
tags: 
  - "assetmanager"
  - "centos"
  - "mysql"
  - "nginx"
  - "percona"
  - "php"
  - "php-fpm"
  - "snipe-it"
image:
  path: /commons/913980_54f3_3.jpg
  alt: "Установка ITSM-системы Snipe-IT"
---

> **Snipe-IT** - open source, кроссплатформенная, многофункциональная система управления ИТ-активами с открытым исходным кодом, построенная с использованием PHP-фреймворка Laravel.

## Подготовка

Обновляем операционную систему, добавляем репозиторий EPEL, устанавливаем софт

```sh
$ sudo yum -y update
$ sudo yum -y install epel-release
$ sudo yum -y install nano wget net-tools
```

## Установка и настройка PHP 7.4

Установим `yum-utils` для инструмента `yum-config-manager`

```sh
$ sudo yum -y install yum-utils
```

Включаем remi-репозиторий для установки php 7.4

```sh
$ sudo rpm -Uvh http://rpms.famillecollet.com/enterprise/remi-release-7.rpm
$ sudo yum-config-manager --enable remi-php74
```

Устанавливаем php 7.4 и некоторые компоненты

```sh
$ sudo yum -y install php php-xml php-soap php-xmlrpc php-mbstring php-json php-gd php-pecl-mcrypt php-common php-fpm php-pdo php-mysqlnd php-imap php-embedded php-ldap php-odbc php-zip php-fileinfo php-process php-opcache php-curl php-intl php-pear php-imagick php-memcache php-pspell php-gettext php-apcu php-pecl-recode php-tidy php-xsl php-pear-CAS
```

Меняем настройки в конфигурационных файлах

```sh
$ sudo sed -i 's/^short_open_tag = .*/short_open_tag = On/' /etc/php.ini
$ sudo sed -i 's/^date.timezone = .*/date.timezone = Europe/Moscow/' /etc/php.ini
```

```sh
$ sudo sed -i 's/^opcache.revalidate_freq= .*/opcache.revalidate_freq=0/' /etc/php.d/10-opcache.ini
```

```sh
$ sudo sed -i 's/^user = .*/user = nginx/' /etc/php-fpm.d/www.conf
$ sudo sed -i 's/^group = .*/group = nginx/' /etc/php-fpm.d/www.conf
$ sudo sed -i 's/^listen = .*/listen = /var/run/php-fpm/php-fpm.sock/' /etc/php-fpm.d/www.conf
$ sudo sed -i 's/^listen.owner = .*/listen.owner = nginx/' /etc/php-fpm.d/www.conf
$ sudo sed -i 's/^listen.group = .*/listen.group = nginx/' /etc/php-fpm.d/www.conf
```

Добавляем php-fpm в автозагрузку

```sh
$ sudo systemctl enable php-fpm
```

## Установка и настройка NGINX

Добавим репозиторий Nginx

```sh
$ sudo nano /etc/yum.repos.d/nginx.repo 
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=0
enabled=1
```

Установка web-сервера Nginx

```sh
$ sudo yum -y install nginx
```

Создадим директории для виртуальных хостов

```sh
$ sudo mkdir /etc/nginx/{sites-available,sites-enabled}
```

Отредактируем основной конфиг

```sh
$ sudo nano /etc/nginx/nginx.conf
events {
    worker_connections  1024;
    use epoll;
    multi_accept on;
}


http {
    server_tokens off;
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    sendfile_max_chunk 128k;
    tcp_nopush     on;
    tcp_nodelay on;

    keepalive_timeout  65;
    reset_timedout_connection on;
    client_header_timeout 3;
    client_body_timeout 5;
    send_timeout 3;
    client_header_buffer_size 2k;
    client_body_buffer_size 256k;
    client_max_body_size 12m;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*.conf;
}
```

Отредактируем конфиг `snipe-it.conf`

```sh
$ sudo nano /etc/nginx/sites-available/snipe-it.conf
server {
    listen 80;
    server_name snipeit.example.com;

    root /opt/snipeit/public/;
    index index.php index.html index.htm;
    access_log /var/log/nginx/snipeit.access.log;
    error_log  /var/log/nginx/snipeit.error.log;

    location / {
        try_files $uri $uri/ /index.php$is_args$args;
    }

    location ~ \.php$ {
        try_files $uri $uri/ =404;
        fastcgi_pass unix:/var/run/php-fpm/php-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
        fastcgi_read_timeout 300;
    }
}
```

Создаем сим линк, чтобы активировать конфиг `snipe-it.conf`

```sh
$ sudo ln -s /etc/nginx/sites-available/snipe-it.conf /etc/nginx/sites-enabled/snipe-it.conf
```

Проверяем конфиг

```sh
$ sudo nginx -t
```

Добавляем Nginx в автозагрузку

```sh
$ sudo systemctl enable nginx
```

## Firewall

Настройка firewall, открываем порты

```sh
$ sudo firewall-cmd --zone=public --add-service={http,https} --permanent
$ sudo firewall-cmd --reload
```

## Percona

Добавим репозиторий Percona

```sh
$ sudo yum -y install https://repo.percona.com/yum/percona-release-latest.noarch.rpm
```

Устанавливаем Percona-Server

```sh
$ sudo yum -y install Percona-Server-server-57
```

Запускаем сервис и добавляем его в автозагрузку

```sh
$ sudo systemctl enable --now mysqld
```

Ищем пароль, который сгенерировался автоматом и меняем его

```sh
$ sudo grep -i password /var/log/mysqld.log
$ sudo mysql_secure_installation
New password:
Re-enter new password:
Change the password for root ? ((Press y|Y for Yes, any other key for No) :
Remove anonymous users? (Press y|Y for Yes, any other key for No) : y
Disallow root login remotely? (Press y|Y for Yes, any other key for No) : y
Remove test database and access to it? (Press y|Y for Yes, any other key for No) : y
Reload privilege tables now? (Press y|Y for Yes, any other key for No) : y
```

## Установка Snipe-IT

Создаем пользователя и базу

```sh
$ mysql -u root -p
> CREATE DATABASE snipeitdb;
> GRANT ALL PRIVILEGES ON snipeitdb.* TO 'snipeituser'@'localhost' IDENTIFIED BY 'password';
> FLUSH PRIVILEGES;
> exit;
```

Создаем директории, скачиваем `snipe-it`

```sh
$ cd
$ sudo mkdir -p /opt/snipeit
$ sudo yum -y install git
$ sudo git clone https://github.com/snipe/snipe-it /opt/snipeit
```

Прописываем настройки подключения к базе

```sh
$ cd /opt/snipeit
$ sudo cp .env.example .env
$ sudo nano /opt/snipeit/.env
APP_DEBUG=false
APP_TIMEZONE='Europe/Moscow'
APP_LOCALE=ru

DB_CONNECTION=mysql
DB_HOST=localhost
DB_DATABASE=snipeitdb
DB_USERNAME=snipeituser
DB_PASSWORD='password'
DB_PREFIX=null
DB_DUMP_PATH='/usr/bin'
DB_CHARSET=utf8mb4
DB_COLLATION=utf8mb4_unicode_ci
```

Настраиваем права доступа

```sh
$ sudo chown -R nginx:nginx /opt/snipeit
$ sudo chmod -R 755 storage
$ sudo chmod -R 755 public/uploads
$ sudo chmod -R 755 bootstrap/cache
```

Устанавливаем `php composer` и запускаем установку Lavarel

```sh
$ cd
$ curl -sS https://getcomposer.org/installer | php
$ sudo mv composer.phar /opt/snipeit
$ sudo yum -y install php-bcmath
$ cd /opt/snipeit
$ sudo php composer.phar install --no-dev --prefer-source
```

Запускаем генерацию App key

```sh
$ sudo php artisan key:generate
Application key [base64:ahNofhfghfghfghbERc9j3SsCXZhZWk=] set successfully.
```

Запускаем Nginx

```sh
$ sudo systemctl reload nginx
```

## SELinux

Настраиваем SELinux

```sh
$ sudo chcon -R -t httpd_sys_rw_content_t /opt/snipeit/storage
$ sudo chcon -R -t httpd_sys_rw_content_t /opt/snipeit/bootstrap/cache
$ sudo setsebool -P httpd_can_network_connect on
$ sudo setsebool -P httpd_can_sendmail on
```