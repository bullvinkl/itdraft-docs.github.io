---
title: "Установка web-сервера Nginx для работы с виртуальными хостами, PHP-FPM в режиме работы Sock, Mysql-сервер MariaDB на Centos 7"
date: "2019-07-15"
categories: 
  - Linux
  - Nginx
tags: 
  - "centos"
  - "mariadb"
  - "mysql"
  - "nginx"
  - "php"
  - "php-fpm"
  - "sock"
  - "virtual-host"
image:
  path: /commons/987847_cbc9_2.jpg
  alt: "Установка Nginx"
---

> **Nginx** — веб-сервер и почтовый прокси-сервер, работающий на Unix-подобных операционных системах

Добавим репозитории EPEL и Remi, необходимое ПО и обновим CentOS

```sh
$ sudo yum install epel-release
$ sudo yum install http://rpms.remirepo.net/enterprise/remi-release-7.rpm
$ sudo yum install wget nano mc zip upzip htop
$ sudo yum install update
```

Добавим репозиторий Nginx.  
Для этого создадим файла с содержимым:

```sh
$ sudo nano /etc/yum.repos.d/nginx.repo
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=0
enabled=1 
```

Добавим репозиторий MariaDB.  
Для этого создадим файла с содержимым:

```sh
$ sudo nano /etc/yum.repos.d/mariadb.repo
[mariadb]
name = MariaDB
baseurl = http://yum.mariadb.org/10.1/centos7-amd64
gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
gpgcheck=1
```

Установим web-сервер Nginx

```sh
$ sudo yum install nginx
```

Запускаем Nginx и добавим его в автозагрузку

```sh
$ sudo systemctl start nginx
$ sudo systemctl enable nginx
```

Откроем порты 80 и 443 в firewall

```sh
$ sudo firewall-cmd --zone=public --permanent --add-service=http
$ sudo firewall-cmd --zone=public --permanent --add-service=https
$ sudo firewall-cmd --reload
```

Установим MariaDB

```sh
$ sudo yum install mariadb-server mariadb
```

Запускаем сервер баз данных и запускаем скрипт настройки

```sh
$ sudo systemctl start mariadb
$ sudo mysql_secure_installation

NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

In order to log into MariaDB to secure it, we'll need the current
password for the root user.  If you've just installed MariaDB, and
you haven't set the root password yet, the password will be blank,
so you should just press enter here.

Enter current password for root (enter for none):
OK, successfully used password, moving on...

Setting the root password ensures that nobody can log into the MariaDB
root user without the proper authorisation.

Set root password? [Y/n]
New password:
Re-enter new password:
Password updated successfully!
Reloading privilege tables..
 ... Success!

By default, a MariaDB installation has an anonymous user, allowing anyone
to log into MariaDB without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.

Remove anonymous users? [Y/n]
 ... Success!

Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n]
 ... Success!

By default, MariaDB comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n]
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n]
 ... Success!

Cleaning up...

All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.

Thanks for using MariaDB!
```

Добавим сервис в автозагрузку

```sh
$ sudo systemctl enable mariadb
```

Отключаем SELinux (не безопасно!)

```sh
$ sudo nano /etc/sysconfig/selinux
...
SELINUX=disabled
...
```

Перзагружаемся

```sh
$ sudo reboot
```

## Установка PHP 7.3, настройка PHP-FPM в режиме работы SOCK

Для начала установим `yum-utils` для инструмента `yum-config-manager`

```sh
$ sudo yum install -y yum-utils
```

Включаем remi-репозиторий для установки `php 7.3`

```sh
$ sudo yum-config-manager --enable remi-php73
```

Устанавливаем `php 7.3` и некоторые компоненты

```sh
$ sudo yum install php php-xml php-soap php-xmlrpc php-mbstring php-json php-gd php-mcrypt php-common php-fpm php-pdo php-mysqlnd php-imap php-embedded php-ldap php-odbc php-zip php-fileinfo php-process php-opcache
```

Проверяем:

```sh
$ sudo  php -v
PHP 7.3.7 (cli) (built: Jul  3 2019 11:30:22) ( NTS )
Copyright (c) 1997-2018 The PHP Group
Zend Engine v3.3.7, Copyright (c) 1998-2018 Zend Technologies
    with Zend OPcache v7.3.7, Copyright (c) 1999-2018, by Zend Technologies
```

Настройка PHP-FPM в режиме работы SOCK.  
Запускаем PHP-FPM и добавляем в автозагрузку

```sh
$ sudo systemctl start php-fpm
$ sudo systemctl enable php-fpm
```

Проверяем, запустился ли он

```sh
$ sudo netstat -tulpn | grep php-fpm
tcp        0      0 127.0.0.1:9000          0.0.0.0:*               LISTEN      3031/php-fpm: maste
```

Если не хватает утилиты `netstat`, установим её

```sh
$ sudo yum install net-tools
```

Настраиваем php-fpm

```sh
$ sudo nano /etc/php.ini
```

Нас интересует параметр `cgi.fix_pathinfo`. По умолчанию он закомментирован и установлен в значение `1`. 
Раскомментируем строчку и изменим значение на `0`.

```
cgi.fix_pathinfo=0
```

Также не забываем изменить временную зону, для этого раскомментируем строчку `date.timezone =` и пропишем необходимую временную зону

```
[Date]
; Defines the default timezone used by the date functions
; http://php.net/date.timezone
date.timezone = Europe/Moscow
```

Включим `short_open_tag` для разрешения короткой формы записи тегов PHP

```
short_open_tag = On
```

Изменим пользователя, от которого будет работать php-fpm. Вместо apache укажем nginx.

```sh
$ sudo nano /etc/php-fpm.d/www.conf
...
user = nginx
group = nginx

listen.owner = nginx
listen.group = nginx
...
```

Находим строчку начинающуюся с `listen` и заменяем её полностью новым значением.

```
;listen = 127.0.0.1:9000
listen = /var/run/php-fpm/php-fpm.sock
```

Перезапускаем php-fpm

```sh
$ sudo systemctl restart php-fpm
```

Почему стоит перейти на unix-сокет?

UDS (unix domain socket), в отличии от коммуникации через стек TCP, имеют значительные преимущества:

- не требуют переключение контекста, UDS используют netisr
- датаграмма UDS записываться напрямую в сокет назначения
- отправка дейтаграммы UDS требует меньше операций (нет контрольных сумм, нет TCP-заголвоков, не производиться маршрутизация)
- у UDS задержка на ~66% меньше и пропускная способность в 7 раз больше TCP

Перезапустим Nginx

```sh
$ sudo systemctl restart nginx
```

После перезапуска Nginx у меня вылезла ошибка

```
nginx: [emerg] module "/usr/lib64/nginx/modules/ngx_http_geoip_module.so" version 1012002 instead of 1014000 in /usr/share/nginx/modules/mod-http-geoip.conf:1
```

Эта ошибка появилась из-за того, что я устанавливал Nginx из стандартного репозитория, а потом подключил официальный репозиторий и обновился.

Чтобы решить эту проблему, удалим все модули из репозитория Nginx:

```sh
$ sudo sudo yum remove nginx-mod*
```

Устанавливаем модули из официального репозитория

```sh
$ sudo sudo yum install nginx-module-*
```

Перезапускаем Nginx

```sh
$ sudo sudo systemctl restart nginx
```

## Настройка NGINX для работы с виртуальными хостами

Создадим необходимые директории для виртуального хоста

```sh
$ sudo mkdir -p  /var/www/example.ru/public_html
$ sudo mkdir /var/www/example.ru/logs
```

В корне создадим html-файл

```sh
$ sudo nano /var/www/example.ru/public_html/index.html
<html>
  <head>
    <title>Sample web page on example.ru website</title>
  </head>
  <body>
    <h1>Nginx server</h1>
    This sample web page confirms that the first Nginx virtual host 
   or server block is working for example.ru
  </body>
</html>
```

Выставляем права на директорию `public_html`

```sh
$ sudo chmod -R 755 /var/www/example.ru/public_html
```

Создадим директории для конфигов виртуальных хостов

```sh
$ sudo mkdir /etc/nginx/sites-available
$ sudo mkdir /etc/nginx/sites-enabled
```

Отредактируем основной конфиг `nginx.conf`

```sh
$ sudo nano /etc/nginx/nginx.conf
```

Перед закрывающейся фигурной скобкой блока `http` добавим строки:

```
include /etc/nginx/sites-enabled/*.conf;
server_names_hash_bucket_size 64;
```

Создаем конфиг-файл нашего виртуального хоста

```sh
$ sudo nano /etc/nginx/sites-available/example.ru.conf
server {
    listen  80;
    server_name example.ru www.example.ru;
	access_log /var/www/example.ru/logs/access.log;
    error_log /var/www/example.ru/logs/error.log;
	root /var/www/example.ru/public_html;
	
location / {
        try_files $uri $uri/ /index.html;
    }
 
      location ~ /\.ht {
        deny all;
    }
 
    location ~ /\. {
        deny all;
        access_log off;
        log_not_found off;
    }
 
    location ~ \.(js|css|png|jpg|gif|swf|ico|pdf|mov|fla|zip|rar)$ {
        try_files $uri =404;
    }
 
    location ~* .php$ {
        include /etc/nginx/fastcgi_params;
        fastcgi_pass unix:/var/run/php-fpm/php-fpm.sock;
        #fastcgi_pass  127.0.0.1:9000; ## Если работаем не по сокетам
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME /var/www/example.ru/public_html$fastcgi_script_name;
    }
}
```

Делаем символическую ссылку. Таким образом проще отключать/подключать новые виртуальные хосты

```sh
$ sudo ln -s /etc/nginx/sites-available/example.ru.conf /etc/nginx/sites-enabled/example.ru.conf
```

И перезапускаем Nginx

```sh
$ sudo systemctl restart nginx
```

Проверяем работу Nginx и PHP

Создаем файл `info.php`

```sh
$ sudo nano /var/www/example.ru/public_html/info.php
<?php phpinfo(); ?>
```

В файле hosts на рабочем ПК прописываем соответствие ip и адреса сайта (как это сделать, можете спросить у меня в комментариях) и убеждаемся что nginx и php работают.