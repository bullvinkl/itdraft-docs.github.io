---
title: "Установка PostgreSQL из исходников, и запуск двух версий на одном сервере в Centos 8"
date: "2020-11-12"
categories: 
  - Linux
  - PostgreSQL
tags: 
  - "postgresql"
image:
  path: /commons/programming-flat-60074276.jpg
  alt: "Установка PostgreSQL из исходников, и запуск двух версий на одном сервере"
---

> **PostgreSQL** — свободная объектно-реляционная система управления базами данных, наиболее развитая из открытых СУБД и являющаяся реальной альтернативой коммерческим базам данных. PostgreSQL базируется на языке SQL.
{: .prompt-tip }

## Подготовка

Устанавливаем необходимые пакеты

```sh
$ sudo dnf -y install gcc make readline-devel zlib-devel systemd-devel
```

Создаем системного пользователя postgres

```sh
$ sudo useradd -M -s -d /var/lib/postgresql postgres
```

Создаем каталог для логов PostgreSQL и назначаем права

```sh
$ sudo mkdir /var/log/postgresql
$ sudo chown -R postgres:postgres /var/log/postgresql
```

## Установка PostgreSQL 9.6 из исходников, запуск на порту 5433

Скачиваем архив PostgreSQL 9.6 в каталог /tmp и разархивируем его

```sh
$ wget https://ftp.postgresql.org/pub/source/v9.6.19/postgresql-9.6.19.tar.gz -P /tmp
$ cd /tmp
$ tar xzf postgresql-9.6.19.tar.gz
$ cd postgresql-9.6.19
```

Запускаем конфигурирование PostgreSQL 9.6

```sh
$ sudo ./configure --with-systemd --prefix=/opt/postgresql/9.6
```

Устанавливаем PostgreSQL 9.6

```sh
$ sudo make
$ sudo make install
```

Создаем каталог для базы PostgreSQL 9.6 и назначаем права

```sh
$ sudo mkdir -p /var/lib/postgresql/9.6/main
$ sudo chown -R postgres:postgres /var/lib/postgresql
```

Переключаемся на пользователя postgres

```sh
$ sudo su - postgres
```

Инициализируем базу и закрываем сессию пользователя postgres

```sh
$ /opt/postgresql/9.6/bin/initdb -D /var/lib/postgresql/9.6/main
$ exit
```

Редактируем конфиг PostgreSQL 9.6, меняем дефолтный порт на 5433

```sh
$ sudo nano /var/lib/postgresql/9.6/main/postgresql.conf
[...]
listen_addresses = '*'
port = 5433
```

Запускаем PostgreSQL 9.6

```sh
$ sudo su - postgres -c "/opt/postgresql/9.6/bin/pg_ctl -D /var/lib/postgresql/9.6/main/ -l /var/log/postgresql/postgresql-9.6.log start"
```

Проверяем, доступен ли порт 5433

```sh
$ ss -nltup | grep 5433
tcp     LISTEN   0        128              0.0.0.0:5433          0.0.0.0:*
tcp     LISTEN   0        128                 [::]:5433             [::]:*
```

Проверяем подключение к СУБД

```sh
$ sudo su - postgres
$ export PGPORT=5433
$ /opt/postgresql/9.6/bin/psql
```

Смотрим версию

```sh
=# SELECT version();
 PostgreSQL 9.6.19 on x86_64-pc-linux-gnu, compiled by gcc (GCC) 8.3.1 20191121 (Red Hat 8.3.1-5), 64-bit
```

Выходим

```sh
=# \q
$ exit
```

Останавливаем PostgreSQL 9.6

```sh
$ sudo su - postgres -c "/opt/postgresql/9.6/bin/pg_ctl -D /var/lib/postgresql/9.6/main/ stop"
```

Для удобства запуска сервиса создадим Systemd Unit для Postgresql 9.6

```sh
$ sudo nano /etc/systemd/system/postgresql-9.6.service
[Unit]
Description=PostgreSQL database server
Documentation=man:postgres(1)

[Service]
Type=notify

User=postgres
Group=postgres

# Location of database directory
Environment=PGDATA=/var/lib/postgresql/9.6/main/

ExecStart=/opt/postgresql/9.6/bin/postmaster -D ${PGDATA}
ExecReload=/bin/kill -HUP $MAINPID
KillMode=mixed
KillSignal=SIGINT

TimeoutSec=0

[Install]
WantedBy=multi-user.target
```

Добавляем сервис в автозагрузку и проверяем статус

```sh
$ sudo systemctl daemon-reload
$ sudo systemctl enable --now postgresql-9.6
$ systemctl status postgresql-9.6
```

Открываем порт 5433 в Firewalld

```sh
$ sudo firewall-cmd --permanent --zone=public --add-port=5433/tcp
$ sudo firewall-cmd --reload
```

## Установка PostgreSQL 11 из исходников, запуск на порту 5432

Скачиваем архив PostgreSQL 11 в каталог /tmp и разархивируем его

```sh
$ wget https://ftp.postgresql.org/pub/source/v11.7/postgresql-11.7.tar.gz -P /tmp
$ cd /tmp
$ tar xzf postgresql-11.7.tar.gz
$ cd postgresql-11.7
```

Запускаем конфигурирование PostgreSQL 11

```sh
$ sudo ./configure --with-systemd --prefix=/opt/postgresql/11
```

Устанавливаем PostgreSQL 11

```sh
$ sudo make
$ sudo make install
```

Создаем каталог для базы PostgreSQL 11 и назначаем права

```sh
$ sudo mkdir -p /var/lib/postgresql/11/main
$ sudo chown -R postgres:postgres /var/lib/postgresql
```

Переключаемся на пользователя postgres

```sh
$ sudo su - postgres
```

Инициализируем базу и закрываем сессию пользователя postgres

```sh
$ /opt/postgresql/11/bin/initdb -D /var/lib/postgresql/11/main
$ exit
```

Редактируем конфиг PostgreSQL 11

```sh
$ sudo nano /var/lib/postgresql/11/main/postgresql.conf
[…]
listen_addresses = '*'
port = 5432
```

Запускаем PostgreSQL 11

```sh
$ sudo su - postgres -c "/opt/postgresql/11/bin/pg_ctl -D /var/lib/postgresql/11/main/ -l /var/log/postgresql/postgresql-11.log start"
```

Проверяем, доступен ли порт 5432

```sh
$ ss -nltup | grep 5432
tcp     LISTEN   0        128              0.0.0.0:5432          0.0.0.0:*
tcp     LISTEN   0        128                 [::]:5432             [::]:*
```

Проверяем подключение к СУБД

```sh
$ sudo su - postgres
$ /opt/postgresql/11/bin/psql
```

Смотрим версию

```sh
=# SELECT version();
PostgreSQL 11.7 on x86_64-pc-linux-gnu, compiled by gcc (GCC) 8.3.1 20191121 (Red Hat 8.3.1-5), 64-bit
```

Выходим

```sh
=# \q
$ exit
```

Останавливаем PostgreSQL 11

```sh
$ sudo su - postgres -c "/opt/postgresql/11/bin/pg_ctl -D /var/lib/postgresql/11/main/ stop"
```

Для удобства запуска сервиса создадим Systemd Unit для Postgresql 11

```sh
$ sudo nano /etc/systemd/system/postgresql-11.service
[Unit]
Description=PostgreSQL database server
Documentation=man:postgres(1)

[Service]
Type=notify

User=postgres
Group=postgres

# Location of database directory
Environment=PGDATA=/var/lib/postgresql/11/main/

ExecStart=/opt/postgresql/11/bin/postmaster -D ${PGDATA}
ExecReload=/bin/kill -HUP $MAINPID
KillMode=mixed
KillSignal=SIGINT

TimeoutSec=0

[Install]
WantedBy=multi-user.target
```

Добавляем сервис в автозагрузку и провряем статус

```sh
$ sudo systemctl daemon-reload
$ sudo systemctl enable --now postgresql-11
$ systemctl status postgresql-11
```

Открываем порт 5432 в Firewalld

```sh
$ sudo firewall-cmd --permanent --zone=public --add-port=5432/tcp
$ sudo firewall-cmd --reload
```
