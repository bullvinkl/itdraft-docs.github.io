---
title: "Резервное копирование и восстановление базы PostgreSQL. Полезные команды"
date: "2018-08-17"
categories: 
  - Database-System
tags: 
  - "backup"
  - "postgresql"
  - "restore"
image:
  path: /commons/1361114_fab1_2.jpg
  alt: "Резервное копирование и восстановление базы PostgreSQL"
---

> **PostgreSQL** — свободная объектно-реляционная система управления базами данных. Существует в реализациях для множества UNIX-подобных платформ.
{: .prompt-tip }

## Резервное копирование базы PostgreSQL (Backup)

```sh
$ pg_dump --host localhost --port 5432 --username "postgres" --role "postgres" --no-password --format tar --blobs --encoding UTF8 --verbose --file /home/backup/postgres.dump "dbname"
```

или можно так

```sh
$ pg_dump -U user dbname > /home/backup/postgres.dump
```

## Восстановление базы PostgreSQL (Restore)

```sh
$ /usr/pgsql-9.6/bin/pg_restore -U postgres -d dbname /home/backup/postgres.dump
```

или можно так

Переключаемся на пользователя postgres

```sh
$ sudo su - postgres
```

Восстанавливаем

```sh
$ psql -h localhost -U postgres -p 5432 dbname < /home/backup/postgres.dump
```

## Полезные команды при работе с PostgreSQL в консоле

### Создать пользователя и базу:

Переключаемся на пользователя postgres и запускаем psql

```sh
$ sudo su - postgres
$ psql
```

Создаем базу `dbname`, создаем пользователя `dbuser` с паролем `pass`, назначаем привилегии и выходим

```sh
=# create database dbname with encoding='UNICODE';
=# create user dbuser with password 'pass';
=# grant all privileges on database dbname to dbuser;
=# \q
```

### Ставим пароль на пользователя postgres

Переключаемся на пользователя `postgres` и запускаем `psql`

```sh
$ sudo su - postgres
$ psql
```

Задаем пароль и выходим

```sh
=# \password
Enter new password: postgres
Enter it again: postgres
=# \q
```

Либо можно так:

```sh
=# ALTER ROLE postgres WITH PASSWORD 'postgres';
=# \q
```

### Перезагрузить конфиг postgresql без перезапуска службы

Переключаемся на пользователя postgres

```sh
$ sudo su - postgres
```

Перезагружаем конфиг

```sh
$ /usr/pgsql-9.6/bin/pg_ctl reload
```

### Еще команды для работы с PostgreSQL

Выбрать базу dbname

```sh
$ psql -d dbname
```

Посмотреть колонки

```sh
=# \d
```

Выводит список пользователей базы данных

```sh
=# \du
```

Выводит список схем

```sh
=# \dn
```

Выводит список баз данных на сервере и показывает их имена, владельцев, кодировку набора символов и права доступа

```sh
=# \l
```
