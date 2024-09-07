---
title: "Создание и восстановление из резервной копии базы PostgreSQL (backup and restore)"
date: "2016-03-01"
categories: 
  - Database-System
tags: 
  - "backup"
  - "postgresql"
  - "restore"
image:
  path: /commons/mockup-2443050_1280.jpg
  alt: "Создание и восстановление из резервной копии базы PostgreSQL"
---

> **PostgreSQL** — свободная объектно-реляционная система управления базами данных. Существует в реализациях для множества UNIX-подобных платформ.
{: .prompt-tip }

Создаем базу, для этого запускаем `psql` под пользователем `postgres`

```sh
$ sudo su - postgres -c psql
```

и вводим следующие команды:

```sh
=# create database dbname with encoding='UNICODE';
=# create user dbuser with password 'dbpass';
=# grant all privileges on database dbname to dbuser;
```

Где
- `dbname` – имя базы данных,
- `dbuser` – имя пользователя,
- `dbpass` – пароль пользователя `dbuser`

Создание резервной копии базы PostgreSQL (Backup):

```sh
$ /usr/pgsql-9.3/bin/pg_dump --username "dbuser" --format custom --blobs --encoding UTF8 --verbose --file "/home/dbname.backup" "dbname"
```

Восстановление из резервной копии базы PostgreSQL (Restore):

```sh
$ /usr/pgsql-9.3/bin/pg_restore -U postgres -d "dbname" "/home/dbname.backup"
```
