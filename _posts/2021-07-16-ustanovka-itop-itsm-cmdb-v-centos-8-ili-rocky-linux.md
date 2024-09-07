---
title: "Установка iTop ITSM & CMDB в Centos 8 или Rocky Linux"
date: "2021-07-16"
categories: 
  - Linux
tags: 
  - "centos"
  - "cmdb"
  - "firewall"
  - "itop"
  - "itsm"
  - "linux"
  - "mariadb"
  - "mysql"
  - "nginx"
  - "percona"
  - "php"
  - "php-fpm"
  - "rocky-linux"
  - "selinux"
image:
  path: /commons/warning01.png
  alt: "Установка iTop ITSM & CMDB"
---

> **iTop (IT Operational Portal)** - это веб-продукт с открытым исходным кодом, предназначенный для автоматизации ИТ-подразделений предприятий и сервис провайдеров. iTop разработан на основе лучших практик ITIL/ITSM и в то же время является достаточно гибким, чтобы адаптироваться к процессам вашей организации.
{: .prompt-tip }

## Подготовка

Устанавливаем пакет утилит для автоматической визуализации графов, т.к. нам понадобится компонент /usr/bin/dot

```sh
$ sudo dnf -y install graphviz
```

Устанавливаем дополнительные утилиты

```sh
$ sudo dnf -y install wget nano unzip dnf-utils policycoreutils-python-utils
```

## Установка web-сервера Nginx

Устанавливаем web-сервер Nginx. Для этого добавляем репозиторий

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

Устанавливаем NGINX, добавляем его в автозагрузку и запускаем его. Проверяем статус, задействован ли порт 80/tcp

```sh
$ sudo dnf -y install nginx
$ sudo systemctl enable --now nginx
$ systemctl status nginx
$ ss -nltup
```

## Настройка Firewall

Разрешаем подключение по портами 80/tcp (http), 443/tcp (https)

```sh
$ sudo firewall-cmd --zone=public --add-service={http,https} --permanent
$ sudo firewall-cmd --reload
```

## Установка php, настройка php-fpm

Добавляем репозиторий Remirepo

```sh
$ sudo dnf -y install https://rpms.remirepo.net/enterprise/remi-release-8.rpm
```

Чистим метаданные, обновляемся

```sh
$ sudo dnf clean metadata
$ sudo dnf -y update
```

Включаем модуль remi-7.4 для установки PHP 7.4 из подключенного репозитория

```sh
$ sudo dnf module list php
$ sudo dnf module enable php:remi-7.4
$ sudo dnf module list php
```

Устанавливаем PHP и необходимые модули

```sh
$ sudo dnf -y install php php-fpm 
$ sudo dnf -y install php-common php-cli php-mysqlnd php-mcrypt php-ldap php-soap php-json php-xml php-gd php-zip
```

Настраиваем PHP

```sh
$ sudo sed -i 's@^short_open_tag = .*@short_open_tag = On@' /etc/php.ini
$ sudo sed -i 's@^date.timezone = .*@date.timezone = Europe/Moscow@' /etc/php.ini

$ sudo sed -i 's@^opcache.revalidate_freq= .*@opcache.revalidate_freq=0@' /etc/php.d/10-opcache.ini
```

Настраиваем PHP-FPM. Меняем владельца в конфиге php-fpm

```sh
$ sudo sed -i 's@^user = .*@user = nginx@' /etc/php-fpm.d/www.conf
$ sudo sed -i 's@^group = .*@group = nginx@' /etc/php-fpm.d/www.conf
$ sudo sed -i 's@^listen.owner = .*@listen.owner = nginx@' /etc/php-fpm.d/www.conf
$ sudo sed -i 's@^listen.group = .*@listen.group = nginx@' /etc/php-fpm.d/www.conf
```

Меняем владельца каталогов (по умолчанию стоит root:apache)

```sh
$ sudo chown -R root:nginx /var/lib/php/session
$ sudo chown -R root:nginx /var/lib/php/opcache
$ sudo chown -R root:nginx /var/lib/php/wsdlcache
```

Добавляем php-fpm в автозагрузку и запускаем сервис. Смотрим статус

```sh
$ sudo systemctl enable --now php-fpm
$ sudo systemctl status php-fpm
```

## Установка iTop, настройка Nginx

Создаем каталог для iTop

```sh
$ sudo mkdir -p /opt/itop
```

[Скачиваем iTop](https://sourceforge.net/projects/itop/files/itop/2.7.4/iTop-2.7.4-7194.zip/download)

Распаковываем его

```sh
$ unzip iTop-2.7.4-7194.zip
```

Перемещаем распакованный каталог web в `/opt/itop` и меняем владельца

```sh
$ sudo mv web /opt/itop/
$ sudo chown -R nginx:nginx /opt/itop/
```

Отключаем дефолтный конфиг Nginx

```sh
$ sudo mv /etc/nginx/conf.d/default.conf /etc/nginx/conf.d/default.disabled
```

Создаем новый конфиг Nginx

```sh
$ sudo nano /etc/nginx/conf.d/itop.conf
server {
    listen 80;
    server_name itop.itdraft.ru;

    root /opt/itop/web/;
    index index.php index.html index.htm;
    access_log /var/log/nginx/itop.access.log;
    error_log  /var/log/nginx/itop.error.log;

    location / {
        try_files $uri $uri/ /index.php$is_args$args;
    }

    location ~ \.php$ {
        try_files $uri $uri/ =404;
        fastcgi_pass unix:/run/php-fpm/www.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
        fastcgi_read_timeout 300;
    }
}
```

Проверяем конфиг Nginx на ошибки и перезапускаем web-сервер

```sh
$ sudo nginx -t
$ sudo systemctl restart nginx
```

## Настройка SELinux

Добавляем разрешающие правила для каталога /opt/itop/web

```sh
$ sudo chcon -R -t httpd_sys_rw_content_t /opt/itop/web
$ sudo setsebool -P httpd_can_network_connect on
$ sudo setsebool -P httpd_can_sendmail on
```

> P.S. В дальнейшем, если будите устанавливать расширения для iTop (extensions), для каталогов с расширениями так же надо добавить разрешающие правила SELinux
{: .prompt-warning }

## Устанавливаем сервер базы данных PerconaDB

Добавляем репозиторий PerconaDB

```sh
$ sudo yum -y install https://repo.percona.com/yum/percona-release-latest.noarch.rpm
```

Выбираем для установки PerconaDB Server 8.0

```sh
$ sudo percona-release setup ps80
```

Чистим кэш, устанавливаем PerconaDB, добавялем сервис в автозагрузку и запускаем его. Проверяем версию

```sh
$ sudo dnf makecache
$ sudo dnf install -y percona-server-server
$ sudo systemctl enable --now mysqld.service
$ mysql -V
```

Смотрим сгенерированный root-пароль

```sh
$ sudo grep "temporary password" /var/log/mysqld.log
```

Запускаем первоначальную настройку PerconaDB

```sh
$ sudo mysql_secure_installation
```

Меняем root-пароль на новый и отвечаем на вопросы

```sh
New pass: 
Change the password for root ? ((Press y|Y for Yes, any other key for No) : N
Remove anonymous users? (Press y|Y for Yes, any other key for No) : Y
Disallow root login remotely? (Press y|Y for Yes, any other key for No) : Y
Remove test database and access to it? (Press y|Y for Yes, any other key for No) : Y
Reload privilege tables now? (Press y|Y for Yes, any other key for No) : Y
All done!
```

## Либо можно установить сервер базы данных MariaDB

Устанавливаем MariaDB, добавляем сервис в автозагрузку и запускаем его. Смотрим статус. Проверяем задействован ли порт 3306/tcp

```sh
$ sudo dnf install mariadb-server mariadb -y
$ sudo systemctl enable --now mariadb
$ systemctl status mariadb
$ ss -nltup
```

Запускаем первоначальную настройку MariaDB

```sh
$ sudo mysql_secure_installation
```

Устанавливаем root-пароль, отвечаем на вопросы

```sh
Enter current password for root (enter for none):
Set root password? [Y/n] Y
New password: 
Re-enter new password: 
Remove anonymous users? [Y/n] Y
Disallow root login remotely? [Y/n] Y
Remove test database and access to it? [Y/n] Y
Reload privilege tables now? [Y/n] Y
Thanks for using MariaDB!
```

## Создаем новую базу и пользователя

Подключаемся к СУБД PerconaDB / MariaDB

```sh
$ mysql -u root -p
```

Смотрим предустановленные базы, проверяем версию

```sh
> show databases;
> select version();
```

Создаем базу и пользователя с паролем для iTop

```sh
> CREATE DATABASE itopdb CHARACTER SET utf8 COLLATE utf8_bin;
> CREATE USER 'itopuser'@'localhost' IDENTIFIED BY 'tqHVy656MX_8RZfa';
> GRANT ALL PRIVILEGES ON itopdb.* to 'itopuser'@'localhost';
> FLUSH PRIVILEGES;
> quit;
```

## Настройка iTop

Далее открываем браузер, переходим по заданному адресу (в данном случае: http://itop.itdraft.ru) и настраиваем iTop. Задаем параметры подключения к базе, какие дополнения ставить и т.д.

В дальнейшем будет рассмотрена интеграция iTop со службой каталогов FreeIPA, сброс пароль админа, настройка e-mail уведомлений и установка расширений как из магазина приложений, так и в ручном режиме.
