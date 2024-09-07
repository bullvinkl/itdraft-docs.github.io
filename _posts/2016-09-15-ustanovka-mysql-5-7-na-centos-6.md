---
title: "Установка MySQL 5.7 в CentOS 6"
date: "2016-09-15"
categories: 
  - Database-System
tags: 
  - "centos"
  - "mysql"
image:
  path: /commons/email-4044165_1280.jpg
  alt: "Установка MySQL 5.7 в CentOS"
---

> **MySQL** - это свободная реляционная система управления базами данных (СУБД), разработанная компанией MySQL AB и позднее поглощенная корпорацией Oracle. Она распространяется под двумя лицензиями: GNU General Public License и собственной коммерческой лицензией.
{: .prompt-tip }

Устанавливаем репозиторий MySQL

```sh
$ wget http://dev.mysql.com/get/mysql57-community-release-el6-7.noarch.rpm
$ sudo rpm -ivh mysql57-community-release-el6-7.noarch.rpm
```

Устанавливаем сервер и клиент MySQL

```sh
$ sudo yum install -y mysql-community-client mysql-community-server
```

Запускаем MySQL

```sh
$ sudo service mysqld start
```

Ищем в log-файле пароль

```sh
$ sudo grep -i temporary /var/log/mysqld.log
```

Подключаемся к MySQL используя пароль из log-файла, и меняем пароль пользователя root

```sh
$ sudo mysql -uroot -p
> SET PASSWORD FOR 'root'@'localhost' = PASSWORD('Yourpassword1!');
```
