---
title: "Обновление PostgreSQL в Linux"
date: "2022-11-22"
categories: 
  - Linux
  - PostgreSQL
tags: 
  - "pg_upgrade"
  - "postgresql"
  - "upgrade"
  - "debian"
image:
  path: /commons/best-free-proxies.png
  alt: "Обновление PostgreSQL в Linux"
---

> **PostgreSQL** — это объектно-реляционная система управления базами данных (ORDBMS), наиболее развитая из открытых СУБД в мире. Она позволяет хранить и управлять крупными объемами данных, обеспечивая высокую надёжность, производительность и масштабируемость.
{: .prompt-tip }

Допустим на сервере установлена PostgeSQL 9.6, требуется обновить её на более новую версию (например при обновлении Zabbix с 3.2 до 6.2)

Установим PostgeSQL 15 из репозитория

```sh
$ sudo apt -y install gnupg2
$ sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
$ wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
$ sudo apt update
$ sudo apt -y install postgresql-15

```

Делаем симлинк

```sh
$ sudo ln -s /usr/lib/postgresql/15/bin/* /usr/sbin/
```

Смотрим, какие еще версии PostgeSQL есть в системе

```sh
$ sudo su - postgres

$ pg_lsclusters
Ver Cluster Port Status Owner    Data directory                Log file
9.4 main    5431 down   postgres /var/lib/postgresql/9.4/main /var/log/postgresql/postgresql-9.4-main.log
9.6 main    5432 up   postgres /var/lib/postgresql/9.6/main /var/log/postgresql/postgresql-9.6-main.log
13  main    5433 up   postgres /var/lib/postgresql/13/main  /var/log/postgresql/postgresql-13-main.log
```

PostgreSQL 9.6 - с данными, не трогаем её, остальные дропаем. Дейстаия выполняются от пользователя postgres

```sh
$ pg_dropcluster --stop 9.4 main
$ pg_dropcluster --stop 13 main
```

Создаем новую БД - PostgreSQL 15

```sh
$ pg_createcluster --start 15 main \
--datadir=/var/lib/postgresql/15/main \
--logfile=/var/log/postgresql/postgresql-15-main.log \
--port=5433
```

Так же можно указать локаль при создании СУБД

```sh
$ pg_createcluster --locale ru_RU.UTF-8 --start 15 main
```

Если надо перезагрузить СУБД воспользуемся командой reload

```sh
$ pg_ctlcluster 15 main reload
```

Смотрим статус, выключаем все СУБД

```sh
$ pg_ctlcluster 15 main status
$ pg_ctlcluster 15 main stop
$ pg_ctlcluster 9.6 main stop

$ pg_lsclusters
Ver Cluster Port Status Owner    Data directory                Log file
9.6 main    5432 down   postgres /var/lib/postgresql/9.6/main /var/log/postgresql/postgresql-9.6-main.log
15  main    5433 down   postgres /var/lib/postgresql/15/main  /var/log/postgresql/postgresql-15-main.log
```

Проверяем готовность СУБД к обновлению

```sh
$ /usr/lib/postgresql/15/bin/pg_upgrade \
--old-datadir=/var/lib/postgresql/9.6/main \
--new-datadir=/var/lib/postgresql/15/main \
--old-bindir=/usr/lib/postgresql/9.6/bin \
--new-bindir=/usr/lib/postgresql/15/bin \
--old-port=5432 \
--new-port=5433 \
--old-options '-c config_file=/etc/postgresql/9.6/main/postgresql.conf' \
--new-options '-c config_file=/etc/postgresql/15/main/postgresql.conf' \
--check
```

Если на этом этапе ошибок не возникло, можно обновлять PostgeSQL

```sh
$ /usr/lib/postgresql/15/bin/pg_upgrade \
--old-datadir=/mnt/data/postgresql/9.6/main \
--new-datadir=/mnt/data/postgresql/15/main \
--old-bindir=/usr/lib/postgresql/9.6/bin \
--new-bindir=/usr/lib/postgresql/15/bin \
--old-port=5432 \
--new-port=5433 \
--old-options '-c config_file=/etc/postgresql/9.6/main/postgresql.conf' \
--new-options '-c config_file=/etc/postgresql/15/main/postgresql.conf'
```

Если возникли ошибки, разбираемся как их устранить. У меня они были из-за подключенных расширений

Запускаем PostgreSQL, выполняем команду vacuumdb, смотрим статус

```sh
$ pg_ctlcluster 15 main start
$ pg_ctlcluster 9.6 main start
/usr/lib/postgresql/15/bin/vacuumdb --all --analyze-in-stages

$ pg_lsclusters
Ver Cluster Port Status Owner    Data directory                Log file
9.6 main    5432 up   postgres /mnt/data/postgresql/9.6/main /var/log/postgresql/postgresql-9.6-main.log
15  main    5433 up   postgres /mnt/data/postgresql/15/main  /var/log/postgresql/postgresql-15-main.log
```

Если требуется поменять порты, останавливаем PostgreSQL

```sh
$ pg_ctlcluster 15 main stop
$ pg_ctlcluster 9.6 main stop
```

от пользователя с правами sudo, правим конфиг `postgresql.conf`

```sh
$ sudo nano /etc/postgresql/15/main/postgresql.conf
...
port = 5432
```

правим конфиг `pg_hba.conf`

```sh
$ sudo nano /etc/postgresql/15/main/pg_hba.conf
...
# Database administrative login by Unix domain socket
local   all             postgres                                peer

# TYPE  DATABASE        USER            ADDRESS                 METHOD

# "local" is for Unix domain socket connections only
local   all             all                                     peer
# IPv4 local connections:
host    all             all             127.0.0.1/32            md5
# IPv6 local connections:
host    all             all             ::1/128                 md5
# Allow replication connections from localhost, by a user with the
# replication privilege.
local   replication     all                                     peer
host    replication     all             127.0.0.1/32            md5
host    replication     all             ::1/128                 md5
```

Запустить БД можно несколькими способами:

```sh
$ sudo systemctl start postgresql@15-main
```

либо от пользователя postgres

```sh
$ sudo su - postges
$ pg_ctlcluster 15 main start
```

Чтобы запустить все БД, установленные на сервере:

```sh
$ sudo systemctl start postgresql
```
