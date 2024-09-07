---
title: "Установка OPCache для повышения производительности PHP в CentOS 7"
date: "2019-08-06"
categories: 
  - Web
tags: 
  - "apache"
  - "centos"
  - "nginx"
  - "php"
  - "opcache"
image:
  path: /commons/1387342_082d_2.jpg
  alt: "Установка OPCache для повышения производительности PHP"
---

> **OPCache** - это расширение PHP, предназначенное для ускорения выполнения PHP-скриптов, путем кэширования компилированного байт-кода. Это позволяет PHP-серверу сохранять готовый результат выполнения скрипта, а не тратить ресурсы на повторное чтение и компиляцию кода при следующих запросах.
> **Как работает OPCache?** PHP открывает файл с кодом, компилирует его, выполняет. Если файлы не меняются, что бы постоянно не выполнять эти действия opCache кэширует результат. Таким образом экономятся ресурсы сервера.
{: .prompt-tip }

## Установка расширения OPCache

Установим репозитории EPEL и REMI

```sh
$ sudo yum update
$ sudo yum install epel-release
$ sudo yum install http://rpms.remirepo.net/enterprise/remi-release-7.rpm  
```

Установим приложение `yum-utils` для последующего выбора версии PHP

```sh
$ sudo yum install yum-utils
```

Выберем версию PHP используя `yum-config-manager`

```sh
$ sudo yum-config-manager --enable remi-php55 # Для PHP 5.5
$ sudo yum-config-manager --enable remi-php56 # Для PHP 5.6
$ sudo yum-config-manager --enable remi-php70 # Для PHP 7.0
$ sudo yum-config-manager --enable remi-php71 # Для PHP 7.1
$ sudo yum-config-manager --enable remi-php72 # Для PHP 7.2
$ sudo yum-config-manager --enable remi-php73 # Для PHP 7.3
```

Теперь установим расширение `Opcache` и проверим версию PHP

```sh
$ sudo yum install php-opcache		
$ php -v
PHP 7.1.31 (cli) (built: Jul 31 2019 09:59:01) ( NTS )
Copyright (c) 1997-2018 The PHP Group
Zend Engine v3.1.0, Copyright (c) 1998-2018 Zend Technologies
    with Zend OPcache v7.1.31, Copyright (c) 1999-2018, by Zend Technologies
```

## Настройка расширения OPCache

Откроем файл настроек `10-opcache.ini`

```sh
$ sudo nano /etc/php.d/10-opcache.ini
```

Пример настроек:

```
opcache.enable_cli = 1
opcache.memory_consumption = 128
opcache.interned_strings_buffer = 8
opcache.max_accelerated_files = 4000
opcache.revalidate_freq = 60
opcache.fast_shutdown = 1
```

Для применения настроек надо перезагрузить web-сервер (смотря какой у вас установлен)

```sh
$ sudo systemctl restart nginx
$ sudo systemctl restart php-fpm
$ sudo systemctl restart httpd
```
