---
title: "MySQL / Zabbix - Table is marked as crashed and should be repaired"
date: "2015-08-01"
categories: 
  - Monitoring-System
  - Database-System
tags: 
  - "centos"
  - "linux"
  - "mysql"
  - "zabbix"
image:
  path: /commons/penguins.png
  alt: "Table is marked as crashed and should be repaired"
---

> **MySQL** - это клиент-серверное программное обеспечение, предназначенное для управления реляционными базами данных. Её исходный код открыт, что означает, что разработчики могут свободно использовать и изменять код.
{: .prompt-tip }

В логах Mysql `/var/log/mysql.log` появилась ошибка:

```
150729 16:45:44 [ERROR] /usr/sbin/mysqld: Table ‘./zabbix/hystory’ is marked as crashed and should be repaired
```

Подключаемся к mysql

```sh
$ mysql -u %username% -p
Enter password:
```

Выбираем базу

```sh
> use zabbix;
```

Восстанавливаем таблицу

```sh
> REPAIR TABLE hystory;
```

Те же действия можно выполнить через phpmyadmin (sql-запрос)

Если в базе много таблиц с ошибкой `Table is marked as crashed and should be repaired`:

Восстанавливаем базу `user_base`

```sh
$ mysqlcheck -uUSER -pPASSWORD  --repair --extended user_base
```

Или восстанавливаем все базы

```sh
$ mysqlcheck -uUSER -pPASSWORD  --repair --extended -A
```
