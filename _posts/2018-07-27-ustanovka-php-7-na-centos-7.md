---
title: "Установка PHP 7 на Centos 7"
date: "2018-07-27"
categories: 
  - Web
  - Development
tags: 
  - "centos"
  - "php"
image:
  path: /commons/1037058_b070_2.jpg
  alt: "Установка PHP 7 на Centos"
---

> **PHP 7** - это семейство языков программирования для веб-разработки, созданное в 2015 году. Новый движок, называемый PQEngine, был выпущен в 2016 году, поддерживая PHP 7 и предшествующие версии. PHP 7 отличается от предыдущих версий своей архитектурой, которая основана на движке Zend Engine III. 
{: .prompt-tip }

Обновляем операционную систему

```sh
$ sudo yum update
```

Добавляем репозиторий REMI

```sh
$ sudo rpm -ivh http://rpms.remirepo.net/enterprise/remi-release-7.rpm
```

Устанавливаем утилиту для работы с репозиториями

```sh
$ sudo yum install yum-utils
```

Подключаем репозиторий для установки `php 7.2`

```sh
$ sudo yum-config-manager --enable remi-php72
```

Устанавливаем `php` и дополнительные библиотеки для него

```sh
$ sudo yum install php php-xml php-soap php-xmlrpc php-mbstring php-json php-gd php-mcrypt php-common php-devel php-fpm php-pdo php-mysqlnd php-imap php-embedded php-ldap php-odbc curl curl-devel
```
