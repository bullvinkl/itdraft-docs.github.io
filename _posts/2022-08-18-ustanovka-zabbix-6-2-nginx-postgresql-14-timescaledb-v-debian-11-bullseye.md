---
title: "Установка Zabbix 6.2 + Nginx + PostgreSQL 14 + TimescaleDB в Debian 11 Bullseye"
date: "2022-08-18"
categories: 
  - Monitoring-System
  - Database-System
tags: 
  - "debian"
  - "nginx"
  - "php-fpm"
  - "postgresql"
  - "timescaledb"
  - "zabbix"
  - "zabbix-agent2"
image:
  path: /commons/penguins.png
  alt: "Установка Zabbix 6.2"
---

> **Zabbix** — свободная система мониторинга и отслеживания статусов разнообразных сервисов компьютерной сети, серверов и сетевого оборудования, написанная Алексеем Владышевым. Для хранения данных используется MySQL, PostgreSQL, SQLite или Oracle Database, веб-интерфейс написан на PHP.
{: .prompt-tip }

## Подготовка

Обновляем ОС, устанавливаем софт

```sh
$ sudo apt update
$ sudo apt -y upgrade
$ sudo apt -y install nano curl bind9-utils telnet wget net-tools traceroute git tcpdump rsync open-vm-tools mlocate htop tar zip unzip  cloud-guest-utils
$ sudo apt -y install gnupg2
```

## Установка Nginx из репозитория

Добавляем ключ репозитория

```sh
$ wget --quiet -O - https://nginx.org/keys/nginx_signing.key | sudo apt-key add -
```

Добавляем репозиторий Nginx

```sh
$ sudo nano /etc/apt/sources.list.d/nginx.list
# NGINX repo
deb https://nginx.org/packages/mainline/debian/ bullseye nginx
deb-src https://nginx.org/packages/mainline/debian bullseye nginx
```

Устанавливаем Nginx

```sh
$ sudo apt update
$ sudo apt install -y nginx
```

## Установка Postgresql 14 из репозитория

Добавляем репозиторий PostgreSQL

```sh
$ sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
```

Добавляем ключ репозитория

```sh
$ wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
```

Устанавливаем Postgresql 14

```sh
$ sudo apt update
$ sudo apt -y install postgresql-14
```

## Установка Zabbix 6.2 из репозитория

Добавляем репозиторий

```sh
$ wget https://repo.zabbix.com/zabbix/6.2/debian/pool/main/z/zabbix-release/zabbix-release_6.2-1+debian11_all.deb
$ sudo dpkg -i zabbix-release_6.2-1+debian11_all.deb
```

Устанавливаем Zabbix для Nginx и Postgresql

```sh
$ sudo apt update
$ sudo apt -y install zabbix-server-pgsql zabbix-frontend-php php7.4-pgsql zabbix-nginx-conf zabbix-sql-scripts zabbix-agent2
```

Почему то в процессе установки обнаружил, что установился apache2, удаляем его

```sh
$ sudo apt -y remove apache2
$ sudo apt -y autoremove
```

## Настройка Postgresql 14

Редактируем конфиг `pg_hba.conf`, включаем авторизацию по паролю для локальных соединений

```sh
$ sudo nano /etc/postgresql/14/main/pg_hba.conf
...
#host    all             all             127.0.0.1/32            scram-sha-256
host    all             all             127.0.0.1/32            md5
# IPv6 local connections:
#host    all             all             ::1/128                 scram-sha-256
host    all             all             ::1/128                 md5
...
```

Перезапускаем сервис

```sh
$ sudo systemctl restart postgresql
```

## Настройка PHP-FPM

Меняем `timezone` и `listen.mode`

```sh
$ sudo nano /etc/zabbix/php-fpm.conf
...
listen.mode = 0777
...
php_value[date.timezone] = Europe/Moscow
```

Перезапускаем сервис

```sh
$ sudo systemctl restart php7.4-fpm
```

## Настройка Nginx

Редактируем конфиг `zabbix.conf`

```sh
$ sudo nano /etc/nginx/conf.d/zabbix.conf
        listen 80;
        listen [::]:80;
        server_name _;
#        listen          8080;
#        server_name     example.com;

        root    /usr/share/zabbix;
        index   index.php;
        client_max_body_size 100M;
...
```

Отключаем дефолтный конфиг

```sh
$ sudo mv /etc/nginx/conf.d/default.conf /etc/nginx/conf.d/default.conf.disabled
```

Перезапускаем сервис

```sh
$ sudo systemctl restart nginx
```

## Настройка PostgreSQL для Zabbix

Создаем пользователя

```sh
$ sudo -u postgres createuser --pwprompt zabbix
Enter password for new role: mysuperpass
```

Создаем базу

```sh
$ sudo -u postgres createdb -O zabbix zabbix
```

Загружаем данные

```sh
$ zcat /usr/share/zabbix-sql-scripts/postgresql/server.sql.gz | sudo -u zabbix psql zabbix
```

Правим конфиг `zabbix_server.conf`, добавляем пароль пользователя

```sh
$ sudo nano /etc/zabbix/zabbix_server.conf
...
DBPassword=mysuperpass
...
```

Перезапускаем сервисы, добавляем их в автозагрузку

```sh
$ sudo systemctl restart zabbix-server zabbix-agent2 nginx php7.4-fpm
$ sudo systemctl enable zabbix-server zabbix-agent2 nginx php7.4-fpm
```

Zabbix server установлен, заходим в web-интерфейс, завершаем установка

```
Default login: Admin
Default pass: zabbix
```

## Установка TimescaleDB

Ставим софт

```sh
$ sudo apt -y install gnupg postgresql-common apt-transport-https lsb-release
```

Добавляем ключ репозитория

```sh
$ wget --quiet -O - https://packagecloud.io/timescale/timescaledb/gpgkey | sudo sh -c "gpg --dearmor > /etc/apt/trusted.gpg.d/timescaledb.gpg"
```

Добавляем репозиторий timescaledb

```sh
$ echo "deb https://packagecloud.io/timescale/timescaledb/debian/ $(lsb_release -c -s) main" | sudo tee /etc/apt/sources.list.d/timescaledb.list
```

Устанавливаем timescaledb

```sh
$ sudo apt update
$ sudo apt -y install timescaledb-2-postgresql-14
```

Тюним конфиг PostgreSQL

```sh
$ sudo timescaledb-tune --quiet --yes
```

Добавляем параметр `shared_preload_libraries = 'timescaledb'` в конфиг Postgresql

```sh
$ echo "shared_preload_libraries = 'timescaledb'" | sudo tee -a /etc/postgresql/14/main/postgresql.conf
```

Перезапускаем Postgresql

```sh
$ sudo systemctl restart postgresql
```

Добавляем расширение в базу

```sh
$ echo "CREATE EXTENSION IF NOT EXISTS timescaledb CASCADE;" | sudo -u postgres psql zabbix
```

Скачиваем дистрибутив zabbix-6.2.1 и распаковываем его

```sh
$ wget https://cdn.zabbix.com/zabbix/sources/stable/6.2/zabbix-6.2.1.tar.gz
$ tar -zxvf zabbix-6.2.1.tar.gz
```

Добавляем данные в базу

```sh
$ cat zabbix-6.2.1/database/postgresql/timescaledb.sql | sudo -u zabbix psql zabbix
```

Что б zabbix server не ругался на версию TimescaleDB, добавляем в самом конце конфига `zabbix_server.conf`

```sh
$ sudo nano /etc/zabbix/zabbix_server.conf
...
AllowUnsupportedDBVersions=1
```

> Иначе в логах zabbix server будет ошибка:  
> 
> Unsupported DB! timescaledb version is 20702 which is higher than maximum of 20699  
> Recommended version should not be higher than TimescaleDB Community Edition 2.6.
{: .prompt-warning }

Перезапускаем сервисы

```sh
$ sudo systemctl restart postgresql zabbix-server
```

## Установка Zabbix Agent 2 на хостах

Добавляем репозиторий Zabbix

```sh
$ wget https://repo.zabbix.com/zabbix/6.2/debian/pool/main/z/zabbix-release/zabbix-release_6.2-1+debian11_all.deb
$ sudo dpkg -i zabbix-release_6.2-1+debian11_all.deb
```

Устанавливаем Zabbix Agent 2

```sh
$ sudo apt update
$ sudo apt -y install zabbix-agent2
```

Правим конфиг `zabbix_agent2.conf`

```sh
$ sudo nano /etc/zabbix/zabbix_agent2.conf
```

Перезапускаем сервис и добавляем его в автозагрузку

```sh
$ sudo systemctl restart zabbix-agent2
$ sudo systemctl enable zabbix-agent2
```
