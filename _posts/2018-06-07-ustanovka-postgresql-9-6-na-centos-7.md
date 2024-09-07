---
title: "Установка PostgreSQL 9.6 в CentOS 7"
date: "2018-06-07"
categories: 
  - Linux
  - PostgreSQL
tags: 
  - "centos"
  - "postgresql"
image:
  path: /commons/1365176_fbbc.jpg
  alt: "Установка PostgreSQL 9.6 в CentOS"
---

> **PostgreSQL** - это объектно-реляционная система управления базами данных (ORDBMS), наиболее развитая из открытых СУБД в мире. Она позволяет хранить и управлять крупными объемами данных, обеспечивая высокую надёжность, производительность и масштабируемость.
{: .prompt-tip }

## Установка PostgreSQL

Добавляем репозиторий PostgreSQL и обновляемся

```sh
$ sudo yum install https://download.postgresql.org/pub/repos/yum/9.6/redhat/rhel-7-x86_64/pgdg-centos96-9.6-3.noarch.rpm
$ sudo yum update
```

Устанавливаем PostgreSQL 9.6

```sh
$ sudo yum install postgresql96 postgresql96-server postgresql96-lib
```

Инициализируем

```sh
$ sudo /usr/pgsql-9.6/bin/postgresql96-setup initdb
```

Добавляем в автозагрузку PostgreSQL и запускаем его

```sh
$ sudo systemctl enable postgresql-9.6
$ sudo systemctl start postgresql-9.6
```

## Настройка PostgreSQL

Открываем доступ к Postgresql, для этого редактируем в файле `postgresql.conf` строку `listen_addresses`

```sh
$ sudo nano /var/lib/pgsql/9.6/data/postgresql.conf
listen_addresses = '*'
```

Разрешаем подключаться к PostgreSQL с заданных ip-адресов, для этого редактируем файл `pg_hba.conf`

```sh
$ sudo nano /var/lib/pgsql/9.6/data/pg_hba.conf
host all all %ip%/32 md5
```

Перезапускаем PostgreSQL

```sh
$ sudo systemctl restart postgresql-9.6
```

Открываем порт `5432` в Firewall

```sh
$ sudo firewall-cmd --permanent --zone=public --add-port=5432/tcp
$ sudo firewall-cmd --reload
```

Ставим пароль на пользователя postgres

```sh
$ sudo su - postgres
$ psql
=# \password
Enter new password: postgres
Enter it again: postgres
=# \q
$ exit
```
