---
title: "Установка OnlyOffice Document Server + PostgreSQL + Nginx на CentOS 8"
date: "2020-07-21"
categories: 
  - Office-System
  - Database-System
  - web
tags: 
  - "centos"
  - "nginx"
  - "onlyoffice"
  - "postgresql"
  - "rabbitmq"
  - "redis"
image:
  path: /commons/910838_84d6.jpg
  alt: "Установка OnlyOffice Document Server"
---

> **OnlyOffice** — офисный пакет с открытым исходным кодом, разработанный компанией Ascensio System SIA с головным офисом в Риге. Решение включает в себя систему для управления документами, проектами, взаимоотношениями с клиентами и электронной почтой.
{: .prompt-tip }

Дополнительные требования:

- PostgreSQL: версия 9.1 или выше
- Nginx: версия 1.3.13 или выше
- Redis
- RabbitMQ

## Подготовка

Подключаем репозиторий Epel и устанавливаем утилиту nano

```sh
$ sudo dnf -y install epel-release nano
```

Отключаем SeLinux

```sh
$ sudo setenforce 0
$ sudo sed -i "s%SELINUX=enforcing%SELINUX=disabled%g" /etc/sysconfig/selinux
```

Проверяем

```sh
$ sestatus
[...]
Current mode:                   permissive
[...]
```

## Установка PostgreSQL 12

Проверяем, какая версия PostgreSQL есть в базовых репозиториях и какая доступна по-умолчанию

```sh
$ dnf module list postgresql
[...]
CentOS-8 - AppStream
Name         Stream   Profiles             Summary
postgresql   9.6      client, server [d]   PostgreSQL server and client module
postgresql   10 [d]   client, server [d]   PostgreSQL server and client module
postgresql   12       client, server [d]   PostgreSQL server and client module

Hint: [d]efault, [e]nabled, [x]disabled, [i]nstalled
```

В репозитории AppStream доступны версии PostgreSQL 9.6, 10(по-умолчанию) и 12

Переключимся на версию PostgreSQL 12 и установим её

```sh
$ sudo dnf -y module enable postgresql:12
$ sudo dnf -y install postgresql-server
```

Инициализируем БД, запустим PostgreSQL server и добавим его в автозагрузку

```sh
$ sudo postgresql-setup --initdb
 * Initializing database in '/var/lib/pgsql/data'
 * Initialized, logs are in /var/lib/pgsql/initdb_postgresql.log

$ sudo systemctl enable --now postgresql
```

Проверяем

```sh
$ ss -nltup | grep 5432
tcp     LISTEN   0        128            127.0.0.1:5432          0.0.0.0:*
tcp     LISTEN   0        128                [::1]:5432             [::]:*
```

Настраиваем авторизацию в PostgreSQL

```sh
$ sudo nano /var/lib/pgsql/data/pg_hba.conf
[...]
host    all             all             127.0.0.1/32            trust
host    all             all             ::1/128                 trust
```

Перезапускаем сервис

```sh
$ sudo systemctl restart postgresql
```

## Создаем базу данных для OnlyOffice Document Server

Переключаемся на пользователя `postgres` и запускаем консольную утилиту `psql`

```sh
$ sudo su – postgres
$ psql
```

Меняем пароль пользователя `postgres`

```sh
=# \password postgres
Enter new password: 
Enter it again:
```

Создадим новую базу `onlyoffice`, пользователя `onlyoffice` с паролем `password`

```sh
=# create database onlyoffice;
CREATE DATABASE
=# create user onlyoffice with password 'password';
CREATE ROLE
=# grant all privileges on database onlyoffice to onlyoffice;
GRANT
=# exit
$ exit
```

## Установка Redis Server

> Redis — резидентная система управления базами данных класса NoSQL с открытым исходным кодом, работающая со структурами данных типа «ключ — значение».

Устанавливаем Redis Server и добавляем его в автозагрузку

```sh
$ sudo dnf -y install redis
$ sudo systemctl enable --now redis
```

Проверяем

```sh
$ ss -nltup | grep 6379
tcp     LISTEN   0        128            127.0.0.1:6379          0.0.0.0:*
```

## Установка и настройка RabbitMQ Server

> RabbitMQ – это приложение для работы с очередями сообщений _(message-queueing)_, еще его называют меседж брокер _(message broker)_ или менеджер очередей _(queue manager)_.

Добавляем репозиторий RabbitMQ

```sh
$ sudo nano /etc/yum.repos.d/rabbitmq_rabbitmq-server.repo
[rabbitmq_rabbitmq-server]
name=rabbitmq_rabbitmq-server
baseurl=https://packagecloud.io/rabbitmq/rabbitmq-server/el/8/$basearch
repo_gpgcheck=1
gpgcheck=0
enabled=1
gpgkey=https://packagecloud.io/rabbitmq/rabbitmq-server/gpgkey
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt
metadata_expire=300

[rabbitmq_rabbitmq-server-source]
name=rabbitmq_rabbitmq-server-source
baseurl=https://packagecloud.io/rabbitmq/rabbitmq-server/el/8/SRPMS
repo_gpgcheck=1
gpgcheck=0
enabled=1
gpgkey=https://packagecloud.io/rabbitmq/rabbitmq-server/gpgkey
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt
metadata_expire=300
```

Устанавливаем RabbitMQ Server

```sh
$ sudo dnf -q makecache -y --disablerepo='*' --enablerepo='rabbitmq_rabbitmq-server'
$ sudo dnf -y install  rabbitmq-server
```

Создадим и отредактируем конфигурационный файл

```sh
$ sudo nano /etc/rabbitmq/rabbitmq-env.conf
export RABBITMQ_NODENAME=rabbit@localhost
export RABBITMQ_NODE_IP_ADDRESS=127.0.0.1
export ERL_EPMD_ADDRESS=127.0.0.1
```

Запускаем RabbitMQ Server и добавляем его в автозагрузку

```sh
$ sudo systemctl enable --now rabbitmq-server
```

Проверяем

```sh
$ ss -nltup | grep 5672
```

Создадим нового пользователя `onlyoffice` с паролем `mypassword` с помощью консольной утилиты `rabbitmqctl`

```sh
$ sudo rabbitmqctl add_user onlyoffice mypassword
Adding user "onlyoffice" ...
$ sudo rabbitmqctl set_user_tags onlyoffice administrator
Setting tags for user "onlyoffice" to [administrator] ...
$ sudo rabbitmqctl set_permissions -p / onlyoffice ".*" ".*" ".*"
Setting permissions for user "onlyoffice" in vhost "/" ...
```

Проверяем, появился ли наш пользователь

```sh
$ sudo rabbitmqctl list_users
Listing users ...
user    tags
onlyoffice      [administrator]
guest   [administrator]
```

## Установка Nginx

Устанавливаем утилиту dnf-utils

```sh
$ sudo dnf -y install dnf-utils
```

Добавляем репозиторий Nginx

```sh
$ sudo nano /etc/yum.repos.d/nginx.repo
[nginx-stable]
name=nginx stable repo
baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=1
enabled=1
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true

[nginx-mainline]
name=nginx mainline repo
baseurl=http://nginx.org/packages/mainline/centos/$releasever/$basearch/
gpgcheck=1
enabled=0
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true
```

По умолчанию будет использоваться стабильная версия. Если нужна основная версия `mainline`, переключаемся

```sh
$ sudo dnf config-manager --set-enabled nginx-mainline
```

Устанавливаем Nginx

```sh
$ sudo dnf -y install nginx
```

## Установка и настройка OnlyOffice Document Server

Добавляем репозиторий OnlyOffice Document Server

```sh
$ sudo dnf -y install https://download.onlyoffice.com/repo/centos/main/noarch/onlyoffice-repo.noarch.rpm
```

Устанавливаем OnlyOffice Document Server

```sh
$ sudo dnf -y install onlyoffice-documentserver
```

Добавляем сервисы `supervisord` и `nginx` в автозагрузку и запускаем их

```sh
$ sudo systemctl enable --now supervisord
$ sudo systemctl enable --now nginx
```

## Настройка OnlyOffice Document Server

Запускаем скрипт конфигурирования `documentserver-configure.sh`

```sh
$ sudo bash documentserver-configure.sh
```

Скрипт предложит указать параметры подключения к PostgreSQL, Redis и RabbitMQ:

```sh
Configuring database access...
Host: localhost
Database name: onlyoffice
User: onlyoffice
Password: password
Trying to establish PostgreSQL connection... OK
Installing PostgreSQL database... OK
Configuring redis access...
Host: localhost

Trying to establish redis connection... OK
Configuring AMQP access...
Host: localhost
User: onlyoffice
Password: mypassword
Trying to establish AMQP connection... OK
Restarting services... OK
```

## Настройка Firewall

Открываем порт 80/tcp (http) и перезагружаем правила

```sh
$ sudo firewall-cmd --zone=public --add-port=80/tcp --permanent
$ sudo firewall-cmd --reload
```

Проверяем

```sh
$ sudo firewall-cmd --list-all
```

Запускаем браузер и переходим по адресу: http://%ip%

![OnlyOffice Document Server](/assets/img/posts/2020/07/21/onlyoffice.png "OnlyOffice Document Server"){: w="300" }
_Проверка_
