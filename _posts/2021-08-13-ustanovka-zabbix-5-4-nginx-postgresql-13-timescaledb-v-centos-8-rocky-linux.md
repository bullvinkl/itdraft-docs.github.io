---
title: "Установка Zabbix 5.4, Nginx, PostgreSQL 13 + TimescaleDB в Centos 8 / Rocky Linux"
date: "2021-08-13"
categories: 
  - Linux
  - Zabbix
  - PostgreSQL
tags: 
  - "centos"
  - "linux"
  - "nginx"
  - "postgresql"
  - "rocky-linux"
  - "timescaledb"
  - "zabbix"
  - "firewall"
  - "selinux"
image:
  path: /commons/computer2.jpg
  alt: ""
---

> **Zabbix** — свободная система мониторинга и отслеживания статусов разнообразных сервисов компьютерной сети, серверов и сетевого оборудования.
> **TimescaleDB** — это расширение PostgreSQL для работы с временными рядами (time series). Временные ряды можно хранить в PostgreSQL и просто так, но TimescaleDB обеспечивает большую производительность на том же железе.
{: .prompt-tip }

## Установка Zabbix и Nginx

Добавляем репозиторий Zabbix

```sh
$ sudo rpm -Uvh https://repo.zabbix.com/zabbix/5.4/rhel/8/x86_64/zabbix-release-5.4-1.el8.noarch.rpm
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

По умолчанию будет использоваться стабильная версия. Если нужна основная версия(mainline), переключаемся

```sh
$ sudo dnf config-manager --set-enabled nginx-mainline
```

Удаляем все метаданные

```sh
$ sudo dnf clean all
```

Устанавливаем Zabbix для БД PostgreSQL и Nginx

```sh
$ sudo dnf install zabbix-server-pgsql zabbix-web-pgsql zabbix-nginx-conf zabbix-sql-scripts zabbix-agent
```

## Установка PostgreSQL 13

Отключаем модуль PostgreSQL в предустановленном по умолчанию репозитории AppStream

```sh
$ sudo dnf -qy module disable postgresql
```

Проверяем

```sh
$ sudo dnf module list postgresql
CentOS-8 - AppStream Local
Name                          Stream                       Profiles                             Summary                                              
postgresql                    9.6 [x]                      client, server [d]                   PostgreSQL server and client module                  
postgresql                    10 [d][x]                    client, server [d]                   PostgreSQL server and client module                  
postgresql                    12 [x]                       client, server [d]                   PostgreSQL server and client module                  
postgresql                    13 [x]                       client, server [d]                   PostgreSQL server and client module
```

Добавляем репозиторий PostgreSQL

```sh
$ sudo dnf -y install https://download.postgresql.org/pub/repos/yum/reporpms/EL-8-x86_64/pgdg-redhat-repo-latest.noarch.rpm
```

Устанавливаем PostgreSQL 13

```sh
$ sudo dnf -y install postgresql13 postgresql13-server
```

Инициализируем базу

```sh
$ sudo /usr/pgsql-13/bin/postgresql-13-setup initdb
Initializing database … OK
```

Основной конфиг PostgreSQL расположен: `/var/lib/pgsql/13/data/postgresql.conf`

Запускаем PostgreSQL и добавляем сервис в автозагрузку

```sh
$ sudo systemctl enable --now postgresql-13
```

Проверяем статус

```sh
$ systemctl status postgresql-13
```

Устанавливаем пароль для пользователя postgres

```sh
$ sudo su - postgres 
$ psql -c "alter user postgres with password 'mysuperpassword'"
ALTER ROLE
$ exit
```

## Установка TimescaleDB

Добавляем репозиторий TimescaleDB

```sh
$ sudo nano /etc/yum.repos.d/timescale_timescaledb.repo
[timescale_timescaledb]
name=timescale_timescaledb
baseurl=https://packagecloud.io/timescale/timescaledb/el/8/$basearch
repo_gpgcheck=1
gpgcheck=0
enabled=1
gpgkey=https://packagecloud.io/timescale/timescaledb/gpgkey
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt
metadata_expire=300
```

Устанавливаем утилиты

```sh
$ sudo yum install pygpgme yum-utils
```

Устанавливаем timescaledb

```sh
$ sudo yum install timescaledb-2-postgresql-13
```

Останавливаем PostgreSQL

```sh
$ sudo systemctl stop postgresql-13
```

Тюним PostgreSQL (от рута)

```sh
$ sudo su -c 'timescaledb-tune --pg-config=/usr/pgsql-13/bin/pg_config'
```

Запускаем PostgreSQL

```sh
$ sudo systemctl start postgresql-13
```

## Настройка Zabbix и Nginx

Переключаемся на пользователя root

```sh
$ sudo su
```

Создаем пользователя БД для Zabbix

```sh
# sudo -u postgres createuser --pwprompt zabbix
Enter password for new role: zabbixpasswd
Enter it again: zabbixpasswd
```

Создадим БД для Zabbix

```sh
# sudo -u postgres createdb -O zabbix zabbix
```

Импортируем начальную схему и данные

```sh
# zcat /usr/share/doc/zabbix-sql-scripts/postgresql/create.sql.gz | sudo -u zabbix psql zabbix
```

Подключаем расширение timescaledb

```sh
# echo "CREATE EXTENSION IF NOT EXISTS timescaledb CASCADE;" | sudo -u postgres psql zabbix
```

Скачиваем исходники Zabbix и распаковываем архив

```sh
# cd /tmp
# wget https://cdn.zabbix.com/zabbix/sources/stable/5.4/zabbix-5.4.3.tar.gz

# tar -zxvf zabbix-5.4.3.tar.gz
```

Импортируем схему и данные для расширения timescaledb в PostgreSQL

```sh
# cat /tmp/zabbix-5.4.3/database/postgresql/timescaledb.sql | sudo -u zabbix psql zabbix
Creating home directory for zabbix.
NOTICE:  PostgreSQL version 13.3 is valid
NOTICE:  TimescaleDB extension is detected
NOTICE:  TimescaleDB version 2.4.0 is valid
NOTICE:  TimescaleDB is configured successfully
DO
```

Редактируем конфиг PostgreSQL

```sh
$ sudo nano /var/lib/pgsql/13/data/pg_hba.conf
...
#host    all             all             127.0.0.1/32            scram-sha-256
host    zabbix          zabbix          127.0.0.1/32            md5
```

Перезапускаем сервис

```sh
$ sudo systemctl restart postgresql-13
```

Настраиваем подключение Zabbix к PostgreSQL

```sh
$ sudo nano /etc/zabbix/zabbix_server.conf
DBHost=127.0.0.1
DBName=zabbix
DBUser=zabbix
DBPassword=zabbixpasswd
```

Настраиваем Nginx

```sh
$ sudo nano /etc/nginx/conf.d/zabbix.conf
        listen          80;
        server_name     zabbix.example.ru;
...
```

Перезапускаем сервисы и добавляем их в автозагрузку

```sh
$ sudo systemctl restart zabbix-server zabbix-agent nginx php-fpm
$ sudo systemctl enable zabbix-server zabbix-agent nginx php-fpm
```

## Настройка Firewall и SELinux

Открываем порты 80/443

```sh
$ sudo firewall-cmd --zone=public --add-port=80/tcp --permanent
$ sudo firewall-cmd --zone=public --add-port=443/tcp --permanent
$ sudo firewall-cmd --reload
```

Выполним следующие команды, чтобы предоставить веб-интерфейсу Zabbix разрешение на соединение с сервером

```sh
$ sudo setsebool -P httpd_can_connect_zabbix on
```

Также нужно предоставить веб-интерфейсу Zabbix разрешение на соединение с базой данных

```sh
$ sudo setsebool -P httpd_can_network_connect_db on
```

Скачиваем готовый модуль для настройки SELinux

```sh
$ cd /tmp
$ wget https://support.zabbix.com/secure/attachment/53320/zabbix_server_add.te

$ checkmodule -M -m -o zabbix_server_add.mod zabbix_server_add.te
$ semodule_package -m zabbix_server_add.mod -o zabbix_server_add.pp
$ semodule -i zabbix_server_add.pp
```

Создаем свой модуль. Для того, чтобы это получилось, нужно хотя бы один раз неудачно запустить zabbix server с включенным selinux.

```sh
$ sudo ausearch -c 'zabbix_server' --raw | audit2allow -M my-zabbixserver
$ sudo semodule -X 300 -i my-zabbixserver.pp
```

После этого zabbix-server должен запуститься с включенным SeLinux

```sh
$ sudo systemctl restart zabbix-server
```

Далее переходим в web-интерфейс и завершаем настройку Zabbix
