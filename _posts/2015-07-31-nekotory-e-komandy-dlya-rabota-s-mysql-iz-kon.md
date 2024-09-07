---
title: "Некоторые команды для работа с MySQL из консоли"
date: "2015-07-31"
categories: 
  - Database-System
tags: 
  - "linux"
  - "mysql"
image:
  path: /commons/programming-flat-60074276.jpg
  alt: "команды для работа с MySQL из консоли"
---

> **MySQL** - это клиент-серверное программное обеспечение, предназначенное для управления реляционными базами данных. Её исходный код открыт, что означает, что разработчики могут свободно использовать и изменять код.
{: .prompt-tip }

Создать базу данных, пользователя и выставить привилегии

```sh
$ mysql -u root -p
> CREATE DATABASE postfix;
> CREATE USER 'postfix'@'localhost' IDENTIFIED BY 'password';
> GRANT ALL PRIVILEGES ON `postfix`.* TO 'postfix'@'localhost';
> quit;
```

Выбрать базу `%database%`

```sh
> use %database%;
```

Показать все базы

```sh
> show databases;
```

Показать таблицы в базе `%database%`

```sh
> show tables from %database%;
```

Показать 50 строк в таблице `%table%`

```sh
> select * from %table% limit 50;
```

Работа с данными:

```sh
> insert into access values('','');
> DELETE FROM `users` WHERE `id`='115' LIMIT 1;
```
