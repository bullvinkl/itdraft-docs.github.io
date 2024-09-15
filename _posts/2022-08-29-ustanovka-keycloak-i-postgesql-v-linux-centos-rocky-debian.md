---
title: "Установка Keycloak и PostgeSQL в Linux (Centos, Rocky, Debian)"
date: "2022-08-29"
categories:
  - Directory-Service
  - Database-System
tags:
  - almalinux
  - centos
  - debian
  - firewall
  - keycloak
  - linux
  - postgresql
  - rocky-linux
  - nginx
  - java
image:
  path: /commons/open-source-proprietary.png
  alt: "Установка Keycloak и PostgeSQL в Linux (Centos, Rocky, Debian)"
---

> **Keycloak** - продукт с открытым кодом для реализации single sign-on с возможностью управления доступом, нацелен на современные применения и сервисы. По состоянию на 2018 год, этот проект сообщества JBoss находится под управлением Red Hat которые используют его как upstream проект для своего продукта RH-SSO
{: .prompt-tip }

## Подготовка к установке Keycloak

Добавляем пользователя и группу
```sh
$ sudo groupadd -r keycloak$ sudo useradd -m -d /var/lib/keycloak -s /sbin/nologin -r -g keycloak keycloak
```

Создаем каталог, скачиваем дистрибутив
```sh
$ sudo mkdir -p /opt/keycloak
$ sudo wget https://github.com/keycloak/keycloak/releases/download/19.0.1/keycloak-19.0.1.zip -P /opt/keycloak
```

Распаковываем, назначаем права
```sh
$ sudo unzip /opt/keycloak/keycloak-19.0.1.zip -d /opt/keycloak
$ cd /opt
$ sudo chown -R keycloak. keycloak
$ sudo chmod o+x /opt/keycloak/keycloak-19.0.1/bin/
```

## Установка OpenJDK

Устанавливаем `OpenJDK` в RHEL-like дистрибутивах
```sh
$ sudo dnf -y install java-11-openjdk
```

Установка OpenJDK в Debian-like дистрибутивах
```sh
$ sudo apt -y install openjdk-11-jdk
```

Проверяем
```sh
$ java -version
```

## Установка PostgreSQL 14 в RHEL-like (Centos 8-9, Rocky 8-9, AlmaLinux 8-9)

Отключаем модуль PostgreSQL
```sh
$ sudo dnf -qy module disable postgresql
```

Добавляем репозиторий PostgreSQL для RHEL-like 9
```sh
$ sudo dnf -y install https://download.postgresql.org/pub/repos/yum/reporpms/EL-9-x86_64/pgdg-redhat-repo-latest.noarch.rpm
```

Либо добавляем репозиторий PostgreSQL для RHEL-like 8
```sh
$ sudo dnf -y install https://download.postgresql.org/pub/repos/yum/reporpms/EL-8-x86_64/pgdg-redhat-repo-latest.noarch.rpm
```

Устанавливаем PostgreSQL 14
```sh
$ sudo dnf -y install postgresql14-server postgresql14
```

Инициализируем базу PostgreSQL
```sh
$ sudo /usr/pgsql-14/bin/postgresql-14-setup initdb
```

Добавляем сервис в автозагрузку и запускаем
```sh
$ sudo systemctl enable postgresql-14
$ sudo systemctl start postgresql-14
```

Проверяем
```sh
$ systemctl status postgresql-14
```

## Установка PostgreSQL в Debian

Добавляем репозиторий
```sh
$ sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
```

Устанавливаем утилиту `gnupg2` и добавляем ключ репозитория
```sh
$ sudo apt -y install gnupg2
$ wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
```

Устанавливаем PostgreSQL 14
```sh
$ sudo apt update
$ sudo apt -y install postgresql-14
```

## Настройка PostgreSQL

Создаем пользователя и базу для Keycloak
```sh
$ sudo -u postgres psql
=# create user keycloak with password 'mysuperpasswd';
=# create database keycloak owner keycloak;
=# grant all privileges on database keycloak to keycloak;
# \q
```

## SSL

Берем готовый SSL-сертификат для вашего домена

Либо генерим самоподписанный сертификат
```sh
$ sudo openssl req -x509 -sha256 -nodes -days 365 -newkey rsa:2048 -keyout /opt/keycloak/keycloak-19.0.1/conf/server.key.pem -out /opt/keycloak/keycloak-19.0.1/conf/server.crt.pem
```

Назначаем права
```sh
$ sudo chown keycloak. /opt/keycloak/keycloak-19.0.1/conf/server*
```

## Настраиваем Keyсolak

Редактируем конфигурационный файл `keycloak.conf`
```sh
$ sudo nano /opt/keycloak/keycloak-19.0.1/conf/keycloak.conf
# The database vendor.
db=postgres

# The username of the database user.
db-username=keycloak

# The password of the database user.
db-password=mysuperpasswd
 
# The full database JDBC URL. If not provided, a default URL is set based on the selected database vendor.
db-url=jdbc:postgresql://localhost/keycloak

# Observability

# If the server should expose metrics and healthcheck endpoints.
metrics-enabled=true

# HTTP

# The file path to a server certificate or certificate chain in PEM format.
https-certificate-file=/opt/keycloak/keycloak-19.0.1/conf/server.crt.pem

# The file path to a private key in PEM format.
https-certificate-key-file=/opt/keycloak/keycloak-19.0.1/conf/server.key.pem


# Hostname for the Keycloak server.
hostname=keycloak.mydomain.com:8443

#http-enabled=true

# Если надо, включаем логирование в файл
log-console-output=default
log=console,file
log-file=/tmp/keycloak.log
```

## Настройка Firewall для RHEL-like

Открываем порт 8443
```sh
$ sudo firewall-cmd --permanent --zone=public --add-port=8443/tcp
$ sudo firewall-cmd --reload
```

В Debian по-умолчанию Firewall UFW не установлен.

## Запускаем Keycloak

Переходим в каталог
```sh
$ cd /opt/keycloak/keycloak-19.0.1
```

Запускаем keycloak в режиме `developer mode`
```sh
$ sudo bin/kc.sh start-dev
```

Задаем логин / пароль админа
```sh
$ export KEYCLOAK_ADMIN=admin
$ export KEYCLOAK_ADMIN_PASSWORD=passwd
```

Создаем конфиг для прода
```sh
$ sudo bin/kc.sh build
```

Первый запуск и импорт логин-пароль админа в базу
```sh
$ sudo -E bin/kc.sh start
```

После успешного запуска останавливаем процесс (Ctrl+C)

Запускаем Keycloak
```sh
$ sudo bin/kc.sh start --hostname=keycloak.mydomain.com
```

## Создаем Systemd Unit

Создаем файл `keycloak.service`
```sh
$ sudo nano /etc/systemd/system/keycloak.service
[Unit]
Description=Keycloak
After=network.target

[Service]
Type=idle
User=keycloak
Group=keycloak
SuccessExitStatus=0 143
ExecStart=!/opt/keycloak/keycloak-19.0.1/bin/kc.sh start --hostname=keycloak.mydomain.com
TimeoutStartSec=600
TimeoutStopSec=600

[Install]
WantedBy=multi-user.target
```

Запускаем сервис, смотрим статус, добавляем в автозагрузку
```sh
$ sudo systemctl daemon-reload
$ sudo systemctl start keycloak
$ sudo systemctl status keycloak
$ sudo systemctl enable keycloak
```
