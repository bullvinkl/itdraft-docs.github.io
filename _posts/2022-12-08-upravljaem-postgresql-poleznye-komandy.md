---
title: "Управляем PostgreSQL, полезные команды"
date: "2022-12-08"
categories: 
  - Database-System
tags: 
  - "linux"
  - "postgresql"
image:
  path: /commons/mining_decline_2-min.png
  alt: "Управляем PostgreSQL, полезные команды"
---

> **PostgreSQL** — свободная объектно-реляционная система управления базами данных. Существует в реализациях для множества UNIX-подобных платформ.
{: .prompt-tip }

## Управление базами

Переключаемся на пользователя `postgres`

```sh
$ sudo su - postgres
```

Смотрим, список БД

```sh
$ pg_lsclusters
Ver Cluster Port Status        Owner    Data directory                Log file
15  main    5432 down,recovery postgres /var/lib/postgresql/15/main   /var/log/postgresql/postgresql-15-main.log
15  portal  5433 down,recovery postgres /var/lib/postgresql/15/portal /var/log/postgresql/postgresql-15-portal.log
```

Смотрим статус определенной БД

```sh
$ pg_ctlcluster 15 portal status
pg_ctl: no server running
```

Запускаем определенную БД

```sh
$ pg_ctlcluster 15 portal start
Warning: the cluster will not be running as a systemd service. Consider using systemctl:
  sudo systemctl start postgresql@15-portal
```

Смотрим статус

```sh
$ pg_ctlcluster 15 portal status
pg_ctl: server is running (PID: 1115345)
/usr/lib/postgresql/15/bin/postgres "-D" "/var/lib/postgresql/15/portal" "-c" "config_file=/etc/postgresql/15/portal/postgresql.conf"
```

Перезапустить определенную БД

```sh
$ pg_ctlcluster 15 portal reload
```

Так же БД можно запустить через `systemctl` от пользователя с правами `sudo`

```sh
$ sudo systemctl start postgresql@15-main.service
```

Либо запустить все базы на сервере

```sh
$ sudo systemctl start postgresql
```

## Создание, удаление базы

Переключаемся на пользователя `postgres`

```sh
$ sudo su - postgres
```

Создаем базу `main` (версия PostgreSQL 15), не запуская БД

```sh
$ pg_createcluster 15 main
```

Создаем базу `portal` (версия PostgreSQL 14) и запустить БД

```sh
$ pg_createcluster 14 --start portal
```

Создаем базу `main` (версия PostgreSQL 15), локализация `ru_RU`, запустить БД

```sh
$ pg_createcluster --locale ru_RU.UTF-8 --start 15 main
```

Другие ключи создания БД

```sh
$ pg_createcluster
--datadir=dir
--socketdir=dir
--logfile=path
--port=port
```

Удаляем БД

```sh
$ pg_dropcluster --stop 15 main
```

## Обновление БД

Проверяем готовность СУБД к обновлению

```sh
/usr/lib/postgresql/15/bin/pg_upgrade \
--old-datadir=/var/lib/postgresql/11/main \
--new-datadir=/var/lib/postgresql/15/main \
--old-bindir=/usr/lib/postgresql/11/bin \
--new-bindir=/usr/lib/postgresql/15/bin \
--old-port=5432 \
--new-port=5433 \
--old-options '-c config_file=/etc/postgresql/11/main/postgresql.conf' \
--new-options '-c config_file=/etc/postgresql/15/main/postgresql.conf' \
--check
```

Обновление

```sh
/usr/lib/postgresql/15/bin/pg_upgrade \
--old-datadir=/var/lib/postgresql/11/main \
--new-datadir=/var/lib/postgresql/15/main \
--old-bindir=/usr/lib/postgresql/11/bin \
--new-bindir=/usr/lib/postgresql/15/bin \
--old-port=5432 \
--new-port=5433 \
--old-options '-c config_file=/etc/postgresql/11/main/postgresql.conf' \
--new-options '-c config_file=/etc/postgresql/15/main/postgresql.conf'
```

## Другие действия

Переключаемся на пользователя `postgres`

```sh
$ sudo su - postgres
```

Создаем файл `.pgpass`, в котором хранятся пароли для подключения к БД

```sh
$ cat > ~/.pgpass <<EOS
> 172.16.2.8:5432:replication:postgres:mysuperpass
> EOS
$ chmod 0600 ~/.pgpass
```

Переводим БД в режиме Replica в режим записи (Master)

```sh
$ pg_ctl promote -D /var/lib/postgresql/15/main
```
