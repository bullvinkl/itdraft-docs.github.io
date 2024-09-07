---
title: "Установка PKI-системы EJBCA на Centos 7"
date: "2020-05-20"
categories: 
  - PKI
tags: 
  - "centos"
  - "ejbca"
  - "mysql"
  - "pki"
image:
  path: /commons/programming-flat-60074276.jpg
  alt: ""
---

> **EJBCA** — это OpenSource ПО для создания Certification Authority уровня предприятия. EJBCA используется для создания центра сертификации сертификатов инфраструктуры открытого ключа (PKI)
{: .prompt-tip }

## Подготовка

Устанавливаем софт

```sh
$ sudo yum install -y nano tar unzip java-1.8.0-openjdk-devel ant psmisc mariadb bc patch
```

## Установка СУБД MariaDB

Устанавливаем и запускаем MariaDB, проверяем статус

```sh
$ sudo yum install -y mariadb-server
$ sudo systemctl enable --now mariadb
$ systemctl status mariadb
```

Первоначальная настройка MariaDB, устанавливаем пароль для пользователя `root`

```sh
$ sudo mysql_secure_installation
Enter current password for root (enter for none):
Set root password? [Y/n]
Set root password? [Y/n]
New password: %password
%
Re-enter new password: %password%
Password updated successfully!
Remove anonymous users? [Y/n]
Disallow root login remotely? [Y/n]
Remove test database and access to it? [Y/n]
Reload privilege tables now? [Y/n]
Thanks for using MariaDB!
```

Создаем базу, нового пользователя БД, устанавливаем пароль пользователя

```sh
$ sudo mysql -u root -p
mysql> CREATE DATABASE ejbcatest CHARACTER SET utf8 COLLATE utf8_general_ci;
mysql> GRANT ALL PRIVILEGES ON ejbcatest.* TO 'ejbca'@'localhost' IDENTIFIED BY 'ejbca';
mysql> exit
```

## Установка PKI-системы EJBCS

Создадим нового пользователя `ejbca`, от которого будет работать система EJBCA

```sh
$ sudo useradd -m -U -r -d /opt/ejbca ejbca
```

Задаем пароль пользователю

```sh
$ passwd ejbca
New password:
Retype new password:
```

Добавляем пользователя `ejbca` в группу `wheel`, что бы у него появились права суперпользователя (`sudo`)

```sh
$ sudo usermod -aG wheel ejbca
```

Переключаемся на нашего пользователя, переходим в домашний каталог

```sh
$ sudo su - ejbca
$ cd
```

Скачиваем дистрибутив и распаковываем его в домашний каталог текущего пользователя (`/opt/ejbca`)

```sh
$ wget https://netcologne.dl.sourceforge.net/project/ejbca/ejbca6/ejbca_6_15_2_6/ejbca_ce_6_15_2_6.zip
$ unzip ejbca_ce_6_15_2_6.zip
```

Если создавали другие БД и пользователя, правим логин/пароль в установочном скрипте

```sh
$ nano /opt/ejbca/ejbca_ce_6_15_2_6/bin/extra/ejbca-setup.sh

По-умолчанию:
host: localhost
db: ejbcatest
dbuser: ejbca
dbuserpass: ejbca
```

Запускаем установку EJBCA

```sh
$ ./ejbca_ce_6_15_2_6/bin/extra/ejbca-setup.sh
```

Запуск скрипта надо выполнять от текущего пользователя, не root  
Скрипт `ejbca-setup.sh` надо обязательно запускать из каталога, в который распаковывали EJBCA.

В процессе установки отвечаем на вопросы:

```
This installs the EJBCA PKI
found RedHat/CentOS
EJBCA will be installed as OS user 'ejbca'

please install dependencies with:
yum install tar unzip java-1.8.0-openjdk-devel ant psmisc mariadb bc patch

Please select "Yes" if you did so, but not before
1) Yes
2) No
#? 1

This will destroy your complete EJBCA installation in database ejbcatest
Do you want this?
1) Yes
2) No
#? 1

LAST CHANCE TO STOP THIS
Do you really want to destroy your EJBCA installation in database ejbcatest?
1) Yes
2) No
#? 1

[...]

You can now install the superadmin.p12 keystore, from /opt/ejbca/ejbca_ce_6_15_2_6/p12, in your web browser, using the password xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx, and access EJBCA at https://localhost:8443/ejbca
```

## Настраиваем Firewall

Открываем порты

```sh
$ sudo firewall-cmd --add-port=8443/tcp --permanent
$ sudo firewall-cmd --add-port=8442/tcp --permanent
$ sudo firewall-cmd --add-port=8080/tcp --permanent
$ sudo firewall-cmd --reload
```

## Команды для запуска / остановки сервиса EJBCA

Команды выполняются от пользователя `ejbca`

```sh
Start EJBCA:
$ cd ~
$ nohup wildfly/bin/standalone.sh -b 0.0.0.0 > /dev/null 2> /dev/null &

Stop EJBCA:
$ cd ~
$ ./wildfly/bin/jboss-cli.sh --connect :shutdown

Reload EJBCA:
$ cd ~
$ ./wildfly/bin/jboss-cli.sh --connect :reload
```

## EJBCA как системный сервис в Linux

Создаем Systemd Unit ejbca.service

```sh
$ sudo nano /etc/systemd/system/ejbca.service

[Unit]
Description=EJBCA Server Daemon
After=network-online.target

[Service]
Type=simple
User=ejbca
Group=ejbca
UMask=007
WorkingDirectory=/opt/ejbca
ExecStart=/opt/ejbca/wildfly/bin/standalone.sh -b 0.0.0.0
ExecReload=/opt/ejbca/wildfly/bin/jboss-cli.sh --connect :reload
ExecStop=/opt/ejbca/wildfly/bin/jboss-cli.sh --connect :shutdown
Restart=on-failure
TimeoutStopSec=300

[Install]
WantedBy=multi-user.target
```

Если EJBCA был запущен, останавливаем его (от пользователя `ejbca`)

```sh
$ cd ~ && ./wildfly/bin/jboss-cli.sh --connect :shutdown
```

После создания Systemd Unit, необходимо перезагрузить процесс самого systemd, для подхвата изменений. Затем запускаем сервис и добавляем его в автозагрузку. Проверяем статус

```sh
$ sudo systemctl daemon-reload
$ sudo systemctl enable --now ejbca
$ systemctl status ejbca
```

## Полезная информация

После установка EJBCA система выдает информацию о расположении сертификата `superadmin.p12` и пароля к нему:

```
/opt/ejbca/ejbca_ce_6_15_2_6/p12/superadmin.p12
```

Его надо скачать и импортировать в бразуер, например firefox:

```
Настройки - Приватность и защита - Просмотр сертификатов - Ваши сертификаты - Импортировать
```

так же пароль сертификата `superadmin.p12` можно узнать, выполнив команду

```sh
$ grep "superadmin.password" /opt/ejbca/ejbca-custom/conf/web.properties
```

Админка расположена по адресу:

```
https://localhost:8443/ejbca/adminweb/
```
