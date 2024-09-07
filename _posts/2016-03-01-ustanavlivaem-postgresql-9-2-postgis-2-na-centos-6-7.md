---
title: "Устанавливаем PostgreSQL 9.2 + PostGIS 2 в Centos 6.7"
date: "2016-03-01"
categories: 
  - Database-System
tags: 
  - "centos"
  - "postgis"
  - "postgresql"
image:
  path: /commons/mining_price_down-1-min.png
  alt: "PostgreSQL 9.2 + PostGIS 2"
---

> **PostGIS** - это расширение для реляционной базы данных PostgreSQL, которое добавляет поддержку географических объектов и операций над ними. Также PostGIS обеспечивает поддержку различных форматов географических данных, таких как WKT, Well-Known Text, и WKB, Well-Known Binary, что позволяет легко обмениваться данными с другими системами и инструментами.
{: .prompt-tip }

Устанавливаем утилиту `wget` и текстовый редактор `nano`

```sh
$ sudo yum install wget nano
```

## Установка PostgreSQL

Добавляем репозиторий

```sh
$ sudo wget http://yum.pgrpms.org/9.2/redhat/rhel-6-x86_64/pgdg-centos92-9.2-7.noarch.rpm
$ sudo rpm -ivh pgdg-centos92-9.2-7.noarch.rpm
```

Правим конфигурационный файл репозитория `CentOS-Base.repo`, что бы в дальнейшем установить PostgreSQL из добавленного репозитория

```sh
$ sudo nano /etc/yum.repos.d/CentOS-Base.repo
[base]
...
exclude=postgresql*
...

[updates]
...
exclude=postgresql*
...
```

Устанавливаем PostgreSQL 9.2, инициализируем его и добавляем в автозагрузку

```sh
$ sudo yum install postgresql92 postgresql92-server
$ sudo service postgresql-9.2 initdb
$ sudo chkconfig postgresql-9.2 on
```

Входим в систему под именем пользователя `postgres` и запускаем оболочку `psql`

```sh
$ sudo su - postgres
$ psql
```

Зададим пользователю `postgres` пароль

```sh
=# ALTER ROLE postgres WITH PASSWORD 'postgres';
```

Выходим

```sh
=# \q
$ exit
```

Редактируем файл `pg_hba.conf` и меняем тип идентификации на `md5`. Так же добавляем нашу подсеть, чтобы можно было подключаться к PostgreSQL через клиент PgAdmin

```
$ sudo nano /var/lib/pgsql/9.2/data/pg_hba.conf
...
local all all md5
host all all 127.0.0.1/32 md5
host all all 192.168.1.0/24 md5
```

Редактируем `postgresql.conf`

```
$ sudo nano /var/lib/pgsql/9.2/data/postgresql.conf
listen_addresses = '*'
```

Без этого изменения удаленно подключиться к PostgreSQL не получится

Перезапускаем службу

```sh
$ sudo service postgresql-9.2 restart
```

Редактируем правила файерволла, открываем стандартный порт PostgreSQL. Перезапускаем `iptables`

```sh
$ sudo nano /etc/sysconfig/iptables
-A INPUT -m state --state NEW -m tcp -p tcp --dport 5432 -j ACCEPT

$ sudo service iptables restart
```

Добавляем репозиторий EPEL, чтобы в процессе установки PostGIS подтянулись зависимости

```sh
$ sudo rpm -ivh http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
```

## Установка PostGIS

Ставим PostGIS и дополнительные расширения

```sh
$ sudo yum install postgis2_92 postgresql92-contrib
```

Создаем новую базу данных

```sh
$ sudo su - postgres
$ createdb postgis
$ psql postgis
```

Устанавливаем дополнительные расширения и проверяем

```sh
=# CREATE EXTENSION postgis;
=# SELECT PostGIS_full_version();
```

Добавляем поддержку топологии и ставим дополнительные расширения

```sh
=# CREATE EXTENSION postgis_topology;
=# CREATE EXTENSION fuzzystrmatch;
=# CREATE EXTENSION postgis_tiger_geocoder;
```
