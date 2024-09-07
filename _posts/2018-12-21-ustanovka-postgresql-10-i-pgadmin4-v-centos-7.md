---
title: "Установка PostgreSQL 10 и pgAdmin4 в Centos 7"
date: "2018-12-21"
categories: 
  - Database-System
tags: 
  - "centos"
  - "pgadmin"
  - "postgresql"
image:
  path: /commons/1032198_11a3_4.jpg
  alt: "Установка PostgreSQL 10 и pgAdmin4 в Centos"
---

> **PostgreSQL** — свободная объектно-реляционная система управления базами данных (СУБД).  
> PostgreSQL создана на основе некоммерческой СУБД Postgres, разработанной как open-source проект в Калифорнийском университете в Беркли.
{: .prompt-tip }

## Установка PostgreSQL 10

Добавляем репозиторий и устанавливаем PostgreSQL сервер и клиент

```sh
$ sudo yum install https://download.postgresql.org/pub/repos/yum/10/redhat/rhel-7-x86_64/pgdg-redhat10-10-1.noarch.rpm
$ sudo yum install postgresql10-server postgresql10
```

Директория PostgreSQL: `/var/lib/pgsql/10/data/`

Инициализируем базу

```sh
$ sudo /usr/pgsql-10/bin/postgresql-10-setup initdb
```

Запускаем сервис, добавляем его в автозагрузку и проверяем статус

```sh
$ sudo systemctl start postgresql-10
$ sudo systemctl enable postgresql-10
$ sudo systemctl status postgresql-10
```

Проверим PostgreSQL и зададим пароль для пользователя `postgres`

```sh
$ sudo su - postgres
$ psql

psql (10.0)
Type "help" for help.
postgres=# \password postgres
Enter new password:
Enter it again:
```

## Установка web-клиента pgAdmin 4

Установим софт из репозитория PostgreSQL

```sh
$ sudo yum install pgadmin4
```

Во время установки, из-за зависимостей, будут также установлены следующие два пакета `pgadmin4-web` и `httpd web server`.

Переименовываем конфиг для web-интерфейса pgAdmin

```sh
$ sudo mv /etc/httpd/conf.d/pgadmin4.conf.sample /etc/httpd/conf.d/pgadmin4.conf
```

Приводим файл к виду

```sh
$ sudo nano /etc/httpd/conf.d/pgadmin4.conf
<VirtualHost *:80>
LoadModule wsgi_module modules/mod_wsgi.so
WSGIDaemonProcess pgadmin processes=1 threads=25
WSGIScriptAlias /pgadmin4 /usr/lib/python2.7/site-packages/pgadmin4-web/pgAdmin4.wsgi

<Directory /usr/lib/python2.7/site-packages/pgadmin4-web/>
        WSGIProcessGroup pgadmin
        WSGIApplicationGroup %{GLOBAL}
        <IfModule mod_authz_core.c>
                # Apache 2.4
                Require all granted
        </IfModule>
        <IfModule !mod_authz_core.c>
                # Apache 2.2
                Order Deny,Allow
                Deny from All
                Allow from 127.0.0.1
                Allow from ::1
        </IfModule>
</Directory>
</VirtualHost>
```

Далее создадим каталоги для либ и логов для `pgAdmin4`, и поменяем их владельца

```sh
$ sudo mkdir -p /var/lib/pgadmin4/
$ sudo mkdir -p /var/log/pgadmin4/
$ sudo chown -R apache:apache /var/lib/pgadmin4
$ sudo chown -R apache:apache /var/log/pgadmin4
```

Отредактируем файл конфига `config_distro.py`, добавив строки

```sh
$ sudo nano /usr/lib/python2.7/site-packages/pgadmin4-web/config_distro.py

LOG_FILE = '/var/log/pgadmin4/pgadmin4.log'
SQLITE_PATH = '/var/lib/pgadmin4/pgadmin4.db'
SESSION_DB_PATH = '/var/lib/pgadmin4/sessions'
STORAGE_DIR = '/var/lib/pgadmin4/storage'
```

Создадим учетную запись пользователя, с которой мы будем аутентифицироваться в веб-интерфейсе.

```sh
$ sudo python /usr/lib/python2.7/site-packages/pgadmin4-web/setup.py
NOTE: Configuration authentication for SERVER mode.

Enter the email address and password to use for the initial pgAdmin user account:

Email address:
Password:
Retype password:
pgAdmin 4 - Application Initialisation
======================================
```

Теперь можно набрать в браузере `http://%ip-address%/pgadmin4` чтобы попасть в вэб-интерфейс

![](/assets/img/posts/2018/12/21/PgAdmin4-Login.png){: w="300" }
_Админка_
