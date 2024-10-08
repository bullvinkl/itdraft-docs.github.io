---
title: "Установка MySQL-сервера (MariaDB) в Centos 7"
date: "2018-07-27"
categories: 
  - Database-System
tags: 
  - "centos"
  - "mysql"
image:
  path: /commons/nn_guide-min.png
  alt: "Установка MySQL-сервера"
---

> **MySQL** - это клиент-серверное программное обеспечение, предназначенное для управления реляционными базами данных. Её исходный код открыт, что означает, что разработчики могут свободно использовать и изменять код.
{: .prompt-tip }

Обновляем операционную систему

```sh
$ sudo yum update
```

Ставим MariaDB

```sh
$ sudo yum install mariadb-server mariadb
```

Добавляем сервер в автозагрузку и запускаем его

```sh
$ sudo systemctl enable mariadb.service
$ sudo systemctl start mariadb.service
```

Запускаем встроенный сценарий безопасности

```sh
$ sudo mysql_secure_installation
```

Вначале будет запрошен root-пароль, но т.к. в новой установке его нет, просто жмем Enter  
После этого сценарий предложит создать root-пароль и задаст ряд вопросов.

```sh
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

You already have a root password set, so you can safely answer 'n'.

Change the root password? [Y/n] n
 ... skipping.

By default, a MariaDB installation has an anonymous user, allowing anyone
to log into MariaDB without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.

Remove anonymous users? [Y/n] y
 ... Success!

Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n] y
 ... Success!

By default, MariaDB comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n] y
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n] y
 ... Success!

Cleaning up...

All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.

Thanks for using MariaDB!
```

Пробуем подключиться к БД

```sh
$ sudo mysql -u root -p
```
