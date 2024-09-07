---
title: "Установка PostgreSQL 11 на Debian 10"
date: "2019-08-28"
categories: 
  - Database-System
tags: 
  - "debian"
  - "postgresql"
image:
  path: /commons/1361114_fab1_2.jpg
  alt: "Установка PostgreSQL 11"
---

> **PostgreSQL** — свободная объектно-реляционная система управления базами данных. Существует в реализациях для множества UNIX-подобных платформ, а также для Microsoft Windows. PostgreSQL базируется на языке SQL и поддерживает многие из возможностей стандарта SQL:2011
{: .prompt-tip }

## Установка PostgreSQL

Обновляем операционную систему

```sh
$ sudo apt update
$ sudo apt -y upgrade
```

Импортируем ключ подписи репозитория

```sh
$ sudo apt install -y wget
$ wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
```

Добавляем репозиторий

```sh
$ RELEASE=$(lsb_release -cs)
$ echo "deb http://apt.postgresql.org/pub/repos/apt/ ${RELEASE}"-pgdg main | sudo tee  /etc/apt/sources.list.d/pgdg.list
```

Устанавливаем PostgreSQL

```sh
$ sudo apt update
$ sudo apt -y install postgresql-11
```

Убедимся, что служба запустилась

```sh
$ systemctl status postgresql
 ● postgresql.service - PostgreSQL RDBMS
    Loaded: loaded (/lib/systemd/system/postgresql.service; enabled; vendor preset: enabled)
    Active: active (exited) since Fri 2019-03-29 13:15:54 UTC; 3min 37s ago
  Main PID: 1360 (code=exited, status=0/SUCCESS)
     Tasks: 0 (limit: 1148)
    Memory: 0B
    CGroup: /system.slice/postgresql.service
```

Установим пароль администратора PostgreSQL

```sh
$  sudo su - postgres 
# psql -c "alter user postgres with password '%пароль%'" 
ALTER ROLE
```

## Включаем удаленный доступ к PostgreSQL (необязательно)

По умолчанию доступ к серверу баз данных PostgreSQL осуществляется только с локального хоста.

```sh
$ ss -tunelp | grep 5432
tcp   LISTEN  0  128  127.0.0.1:5432         0.0.0.0:*      users:(("postgres",pid=15785,fd=3)) uid:111 ino:42331 sk:6 <->
```

Отредактируем файл конфигурации PostgreSQL 11

```sh
$ sudo nano /etc/postgresql/11/main/postgresql.conf
```

Раскомментируем строку, что бы можно было подключиться к PostgreSQL с любого IP

```
listen_addresses = '*' # Don't do this if your server is on public network
```

Что бы указать конкретный IP подставьте его вместо звездочки

Перезапустим PostgreSQL

```sh
$ sudo systemctl restart postgresql
```

Проверяем

```sh
$ ss -tunelp | grep 5432
tcp     LISTEN   0        128              0.0.0.0:5432          0.0.0.0:*       uid:108 ino:74999 sk:a <->                                                     
tcp     LISTEN   0        128                 [::]:5432             [::]:*       uid:108 ino:75000 sk:b v6only:1 <->
```

## Настройка аутентификации

Что бы включить аутентификацию по паролю отредактируем файл `pg_hba.conf`

```sh
$ sudo nano /etc/postgresql/11/main/pg_hba.conf
```

Примеры аутентификации

Все локальные пользователи могут подключаться к любой базе данных, используя любое имя пользователя, без пароля

```
# TYPE  DATABASE        USER            ADDRESS                 METHOD
local   all             all                                     trust
```

Любой пользователь из сети 192.168.1.x может подключиться к базе postgres используя метод аутентификации ident

```
# TYPE  DATABASE        USER            ADDRESS                 METHOD
host    postgres        all             192.168.1.0/24         ident
```

Любой пользователь компьютера с ip 192.168.1.10 может подключиться к базе "postgres", если он передаёт правильный пароль

```
# TYPE  DATABASE        USER            ADDRESS                 METHOD
host    postgres        all             192.168.1.10/32        scram-sha-256
```

Разрешено подключение локальных пользователей к базе postgres по паролю

```
# TYPE  DATABASE        USER            ADDRESS                 METHOD
local   postgres        all                                     md5
```

## Некоторые команды для работы с PostgreSQL из консоли

Добавим тестового пользователя базы данных

```sh
$ sudo su - postgres
# createuser test_user1
```

Создадим тестовую базу и сделаем пользователя `test_user1` владельцем этой базы

```
# createdb test_db -O test_user1
```

Зададим пароль для тестового пользователя

```sh
# psql 
psql (11.2 (Debian 11.2-2))
Type "help" for help.
> alter user test_user1 with password '%пароль%';
ALTER ROLE
```

Подключимся к базе test\_db

```sh
# psql -l  | grep test_db
test_db   | test_user1 | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 

# psql test_db
psql (11.2 (Debian 11.2-2))
Type "help" for help.
test_db=# 
```

Создадим таблицу и добавим туда данные

```
test_db=# create table test_table ( id int,first_name text, last_name text ); 
CREATE TABLE
test_db=# insert into test_table (id,first_name,last_name) values (1,'John','Doe'); 
INSERT 0 1
```

Посмотрим данные таблицы

```
test_db=# select * from test_table;
  id | first_name | last_name 
 ----+------------+-----------
   1 | John       | Doe
 (1 row)
test_db=# 
```

Удалим тестовую базу

```
# dropdb test_db
```
