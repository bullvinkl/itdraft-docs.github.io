---
title: "Установка PostgreSQL 13 в CentOS 8"
date: "2020-11-30"
categories: 
  - Linux
  - PostgreSQL
tags: 
  - "centos"
  - "postgresql"
image:
  path: /commons/629418_a5b0-1.jpg
  alt: "Установка PostgreSQL 13"
---

> **PostgreSQL** — свободная объектно-реляционная система управления базами данных. Обновления для этой ветки будут выходить в течение пяти лет до ноября 2025 года.

## Установка PostgreSQL 13

Добавляем репозиторий PostgreSQL

```sh
$ sudo dnf -y install https://download.postgresql.org/pub/repos/yum/reporpms/EL-8-x86_64/pgdg-redhat-repo-latest.noarch.rpm
```

Отключаем модуль PostgreSQL в предустановленно по-умолчанию репозитории AppStream

```sh
$ sudo dnf -qy module disable postgresql
```

Проверяем

```sh
$ sudo dnf module list | grep postgresql
postgresql    9.6 [x]     client, server [d]   PostgreSQL server and client module
postgresql    10 [d][x]   client, server [d]   PostgreSQL server and client module
postgresql    12 [x]      client, server [d]   PostgreSQL server and client module
```

Устанавливаем PostgreSQL 13

```sh
$ sudo dnf -y install postgresql13 postgresql13-server
```

Инициализируем базу

```sh
$ sudo /usr/pgsql-13/bin/postgresql-13-setup initdb
Initializing database … OK
```

Основной конфиг PostgreSQL расположен тут: `/var/lib/pgsql/13/data/postgresql.conf`

```sh
$ ls  /var/lib/pgsql/13/data/
base    pg_commit_ts  pg_ident.conf  pg_notify    pg_snapshots  pg_subtrans  PG_VERSION  postgresql.auto.conf
global  pg_dynshmem   pg_logical     pg_replslot  pg_stat       pg_tblspc    pg_wal      postgresql.conf
log     pg_hba.conf   pg_multixact   pg_serial    pg_stat_tmp   pg_twophase  pg_xact
```

Запускаем PostgreSQL и добавляем сервис в автозагрузку

```sh
$ sudo systemctl enable --now postgresql-13
```

Проверяем статус

```sh
$ systemctl status postgresql-13
```

Устанавливаем пароль для пользователя postgres

```sh
$ sudo su - postgres 
$ psql -c "alter user postgres with password 'StrongDBPassword'"
ALTER ROLE
$ exit
```

## Работа с базой / пользователями

Переключаемся на пользователя postgres

```sh
$ sudo su - postgres
```

Создаем пользователя БД

```sh
$ createuser userdb
```

Переключаемся в PostgreSQL shell

```sh
$ psql
```

Задаем пароль для пользователя БД

```sh
=# ALTER USER userdb WITH ENCRYPTED password 'aaayoupasswdaaa';
```

Создам базу и задаем владельца базы

```sh
=# CREATE DATABASE mybase WITH ENCODING='UTF8' OWNER=userdb;
=# \q
$ exit
```

## Настройка PostgreSQL 13

Настраиваем возможность подключения к БД из др. хоста. Для этого редактируем конфигурационный файл `/var/lib/pgsql/13/data/postgresql.conf` и устанавливаем в качестве параметра `Listen address` ip-адрес сервера, или `*` - для всех сетевых интерфейсов

```sh
$ sudo nano /var/lib/pgsql/13/data/postgresql.conf
...
listen_addresses = '192.168.11.200'
...
```

Настраиваем параметры авторизации

```sh
$ sudo nano /var/lib/pgsql/13/data/pg_hba.conf
...
# Accept from trusted subnet
host all all 192.168.11.0/24 md5
...
```

Перезапускаем PostgreSQL

```sh
$ sudo systemctl restart postgresql-13
```

Тестируем подключение

```sh
$ sudo su - postgres
$ psql -U <dbuser> -h <serverip> -p 5432 <dbname>
```

## Настройка Firewall

Открываем порт 5432

```sh
$ sudo firewall-cmd --zone=public --add-port=5432/tcp --permanent
$ sudo firewall-cmd --reload
```