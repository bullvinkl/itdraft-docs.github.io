---
title: "MySQL - Table is marked as crashed and should be repaired"
date: "2016-03-01"
categories: 
  - Database-System
tags: 
  - "linux"
  - "mysql"
  - "repair"
image:
  path: /commons/mixer_money-min-1.png
  alt: "Table is marked as crashed and should be repaired"
---

> **MySQL** - это клиент-серверное программное обеспечение, предназначенное для управления реляционными базами данных. Её исходный код открыт, что означает, что разработчики могут свободно использовать и изменять код.
{: .prompt-tip }

Просматривая `/var/log/mysql/error.log` обнаруживаем ошибки вида

```
090316 20:55:03 [ERROR] /usr/sbin/mysqld: Table ‘./user_base/table’ is marked as crashed and should be repaired
```

Если битых всего несколько таблиц, то можно выполнить `repair table` из консольного mysql клиента или phpmyadmin при помощи sql запроса:

```sh
> USE user_base
> REPAIR TABLE %tablename%;
```

Если в базе много битых таблиц, то будет проще выполнить команду:

```sh
$ mysqlcheck -uUSER -pPASSWORD --repair --extended user_base
```

Ну а если много битых таблиц, да еще и в большом количестве баз, то восстановление лучше запустить на все базы, командой:

```sh
$ mysqlcheck -uUSER -pPASSWORD --repair --extended -A
```
