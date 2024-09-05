---
title: "Обновление PHP 7.0 до PHP 7.2 в Centos 7"
date: "2018-11-08"
categories: 
  - Linux
tags: 
  - "apache"
  - "centos"
  - "php"
image:
  path: /commons/1003248_1dd0_2.jpg
  alt: "Обновление PHP"
---

> **PHP** (Hypertext PreProcessor, «препроцессор гипертекста») — это скриптовый язык программирования, имеющий открытый исходный код. Изначально создавался для разработки веб-приложений, но в процессе обновлений стал языком общего назначения.

Для обновления `PHP 7.0` до `PHP 7.2` на Centos 7 у нас в операционной системе должен быть установлен репозиторий REMI и утилита для работы с репозиториями yum-utils

```sh
$ sudo rpm -ivh http://rpms.remirepo.net/enterprise/remi-release-7.rpm
$ sudo yum install yum-utils
```

Смотрим, какие модули PHP у нас установлены

```sh
$ sudo yum list installed php*
Загружены модули: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirror.reconn.ru
 * epel: mirror.logol.ru
 * extras: mirror.reconn.ru
 * remi-php71: mirror.reconn.ru
 * remi-safe: mirror.reconn.ru
 * updates: mirror.reconn.ru
Установленные пакеты
php.x86_64                               7.1.10-1.el7.remi                          @remi-php71
php-cli.x86_64                           7.1.10-1.el7.remi                          @remi-php71
php-common.x86_64                        7.1.10-1.el7.remi                          @remi-php71
php-gd.x86_64                            7.1.10-1.el7.remi                          @remi-php71
php-json.x86_64                          7.1.10-1.el7.remi                          @remi-php71
php-ldap.x86_64                          7.1.10-1.el7.remi                          @remi-php71
php-mbstring.x86_64                      7.1.10-1.el7.remi                          @remi-php71
php-mysqlnd.x86_64                       7.1.10-1.el7.remi                          @remi-php71
php-odbc.x86_64                          7.1.10-1.el7.remi                          @remi-php71
php-opcache.x86_64                       7.1.10-1.el7.remi                          @remi-php71
php-pdo.x86_64                           7.1.10-1.el7.remi                          @remi-php71
php-pear.noarch                          1:1.10.5-2.el7.remi                        @remi-php71
php-pecl-zip.x86_64                      1.15.4-1.el7.remi.7.1                      @remi-php71
php-process.x86_64                       7.1.10-1.el7.remi                          @remi-php71
php-soap.x86_64                          7.1.10-1.el7.remi                          @remi-php71
php-xml.x86_64                           7.1.10-1.el7.remi                          @remi-php71
php-xmlrpc.x86_64                        7.1.10-1.el7.remi                          @remi-php71
```

Удаляем их, и переключаем репозиторий REMI на работу с PHP версией 7.2

```sh
$ sudo yum remove php*
$ sudo yum-config-manager --disable remi-php71
$ sudo yum-config-manager --enable remi-php72
```

Устанавливаем `PHP 7.2` и модули, которые у нас были и перезапускаем Apache

```sh
$ sudo yum install php php-xml php-soap php-xmlrpc php-mbstring php-json php-gd php-mcrypt php-common php-fpm php-pdo php-mysqlnd php-imap php-embedded php-ldap php-odbc php-zip php-fileinfo php-process php-opcache
$ sudo systemctl restart httpd
```

## UPD 07.02.2019 обновление до PHP 7.3

Для обновления на `PHP 7.3` удаляем `PHP 7.2`, отключаем `PHP 7.2` через `yum-config-manager` и подключаем `PHP 7.3`

```sh
$ sudo yum remove php*
$ sudo yum-config-manager --disable remi-php72
$ sudo yum-config-manager --enable remi-php73
```

Устанавливаем `PHP 7.3` и модули, которые у нас были и перезапускаем Apache

```sh
$ sudo yum install php php-xml php-soap php-xmlrpc php-mbstring php-json php-gd php-mcrypt php-common php-fpm php-pdo php-mysqlnd php-imap php-embedded php-ldap php-odbc php-zip php-fileinfo php-process php-opcache
$ sudo systemctl restart httpd
```