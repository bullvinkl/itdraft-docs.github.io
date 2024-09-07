---
title: "Установка Zabbix 4.2 и MySQL-сервер Percona 8 на Centos 7"
date: "2019-08-15"
categories: 
  - Monitoring-System
  - Database-System
tags: 
  - "apache"
  - "centos"
  - "mysql"
  - "percona"
  - "zabbix"
image:
  path: /commons/1051548_1085_2.jpg
  alt: "Установка Zabbix 4.2 и Percona 8"
---

> **Zabbix** — свободная система мониторинга и отслеживания статусов разнообразных сервисов компьютерной сети, серверов и сетевого оборудования, написанная Алексеем Владышевым. Для хранения данных используется MySQL, PostgreSQL, SQLite или Oracle Database, веб-интерфейс написан на PHP.
{: .prompt-tip }

## Подготовка к установке Zabbix

Подключаем репозитории EPEl и REMI

```sh
$ sudo yum -y install epel-release
$ sudo yum -y install http://rpms.remirepo.net/enterprise/remi-release-7.rpm
```

Устанавливаем необходимые утилиты

```sh
$ sudo yum -y install htop nano mc wget zip unzip yum-utils
```

Подключаем репозиторий Zabbix и обновляемся

```sh
$ rpm -ivh https://repo.zabbix.com/zabbix/4.2/rhel/7/x86_64/zabbix-release-4.2-1.el7.noarch.rpm
$ sudo yum update
```

Веб-интерфейс Zabbix требует дополнительные пакеты, которые отсутствуют в базовой установке. Необходимо активировать репозиторий опциональных rpm пакетов в системе

```sh
$ sudo yum-config-manager --enable rhel-7-server-optional-rpms
```

## Установка Percona 8

Подключаем репозиторий Percona и переключаемся на версию Percona 8

```sh
$ rpm -ivh https://repo.percona.com/yum/percona-release-latest.noarch.rpm
$ sudo percona-release setup ps80
```

Устанавливаем Percona-server, запускаем его и добавляем в автозагрузку

```sh
$ sudo yum -y install percona-server-server
$ sudo systemctl start mysqld
$ sudo systemctl enable mysqld
```

Смотрим дефолтный root-пароль

```sh
$ grep -i password /var/log/mysqld.log
2019-08-07T10:03:19.576146Z 5 [Note] [MY-010454] [Server] A temporary password is generated for root@localhost: f.pjp.N?S1&q
```

Меняем его

```sh
$ sudo mysql_secure_installation
%your_password%
```

Подключаемся к MySQL, создаем базу, пользователя и задаем пароль

```sh
$ mysql -uroot -p'%your_password%'
> create database zabbix character set utf8 collate utf8_bin;
> create user 'zabbix'@'localhost' IDENTIFIED BY '%your_password%';
> grant all privileges on zabbix.* to 'zabbix'@'localhost';
> quit;
```

## Установка Zabbix

Устанавливаем Zabbix

```
$ sudo yum -y install zabbix-server-mysql zabbix-web-mysql
```

Импортируем изначальную схему и данные сервера на MySQL

```sh
$ zcat /usr/share/doc/zabbix-server-mysql*/create.sql.gz | mysql -uzabbix -p zabbix
```

Возможные ошибки.  
У меня в определенном моменте выскочила ошибка:

> Authentication plugin 'caching_sha2_password' cannot be loaded
{: .prompt-danger }

Решение:  
Подключаемся к базе и выполняем команду

```sh
$ mysql -uroot -p'%your_password%'
> ALTER USER 'zabbix'@'localhost' IDENTIFIED WITH mysql_native_password BY '%your_password%';
> quit;
```

Прописываем настройки подключения к базе в конфигурационном файле Zabbix

```sh
$ sudo nano /etc/zabbix/zabbix_server.conf
DBHost=localhost
DBName=zabbix
DBUser=zabbix
DBPassword=%your_password%
```

Запускаем Zabbix-Сервер и добавляем его в автозагрузку

```sh
$ sudo systemctl start zabbix-server
$ sudo systemctl enable zabbix-server
```

Конфигурация Apache для Zabbix веб-интерфейса располагается в /etc/httpd/conf.d/zabbix.conf. В ней надо раскоментировать и прописать наше значение date.timezone

```sh
$ sudo nano /etc/httpd/conf.d/zabbix.conf
...
php_value date.timezone Europe/Moscow
...
```

Перезапускаем web-сервер Apache

```sh
$ sudo systemctl restart httpd
```

Открываем в файерволле порты 80 и 443

```sh
$ sudo firewall-cmd --zone=public --permanent --add-port=443/tcp
$ sudo firewall-cmd --zone=public --permanent --add-port=80/tcp
$ sudo firewall-cmd --reload
```

Теперь можно завершать установку Zabbix через вэб-интерфейс. Для этого открываем браузер и переходим: `http://%ip-adress%/zabbix`

## Настройка SELinux

Если состояние SELinux в принудительном режиме, необходимо выполнить следующие команды, чтобы разрешить соединения между Zabbix веб-интерфейсом и сервером:

```sh
$ sudo setsebool -P httpd_can_connect_zabbix on
$ sudo setsebool -P httpd_can_network_connect_db on
```

Возможные ошибки.  
При запуске zabbix-сервера в логах выскакивает ошибка:

> cannot start alert manager service: Cannot bind socket to "/var/run/zabbix/zabbix_server_alerter.sock"
{: .prompt-danger }

Это связано с SELinix, отключаем его:

```sh
$ sudo setenforce 0
$ sudo nano /etc/selinux/config
SELINUX=disabled
```

UPD: На одном из форумов нашел решение, но не проверял его

```sh
$ sudo grep AVC /var/log/audit/audit.log* | audit2allow -M systemd-allow; semodule -i systemd-allow.pp
```

## Установка Zabbix агента

Устанавливаем Zabbix-агент, запускаем его и добавляем в автозагрузку

```sh
$ sudo yum -y install zabbix-agent
$ sudo systemctl start zabbix-agent
$ sudo systemctl enable zabbix-agent
```

Админский доступ в Zabbix по-умолчанию:

> Login: Admin  
> Password: zabbix
{: .prompt-info }
