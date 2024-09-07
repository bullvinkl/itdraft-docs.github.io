---
title: "Установка Baculum на Centos 7"
date: "2019-12-05"
categories: 
  - Backup-System
tags: 
  - "backup"
  - "bacula"
  - "baculum"
  - "centos"
image:
  path: /commons/629418_a5b0-1.jpg
  alt: "Установка Baculum"
---

> **Baculum** - это веб-интерфейс для управления системой Bacula, обеспечивая удобство и доступность для администрирования и мониторинга. Он является частью проекта Bacula начиная с версии 7.0 и позднее.
{: .prompt-tip }

Скачиваем необходимые пакеты

```sh
$ wget https://bacula.org/downloads/baculum/stable/centos/baculum-common-9.4.4-1.el7.noarch.rpm
$ wget https://bacula.org/downloads/baculum/stable/centos/baculum-web-9.4.4-1.el7.noarch.rpm
$ wget https://bacula.org/downloads/baculum/stable/centos/baculum-web-httpd-9.4.4-1.el7.noarch.rpm
$ wget https://bacula.org/downloads/baculum/stable/centos/baculum-api-9.4.4-1.el7.noarch.rpm
$ wget https://bacula.org/downloads/baculum/stable/centos/baculum-api-httpd-9.4.4-1.el7.noarch.rpm
```

Устанавливаем их

```sh
$ sudo yum localinstall baculum-*
```

Добавляем некоторые sudo-права для пользователя Apache

```sh
$ sudo nano /etc/sudoers.d/baculum
#In case default Apache user:
Defaults:apache !requiretty
apache  ALL=NOPASSWD:  /usr/bin/bconsole
apache  ALL=NOPASSWD:  /usr/bin/bdirjson
apache  ALL=NOPASSWD:  /usr/bin/bsdjson
apache  ALL=NOPASSWD:  /usr/bin/bfdjson
apache  ALL=NOPASSWD:  /usr/bin/bbconsjson
```

Меняем права на каталог

```sh
$ sudo chown -R apache /opt/bacula/etc
```

Перезапускаем сервис `rsyslog`

```sh
$ sudo  systemctl restart rsyslog
```

Добавляем запускаем сервис `apache` и добавляем в автозагрузку

```sh
$ sudo systemctl enable httpd
$ sudo systemctl restart httpd
```

Открываем порты

```sh
$ sudo firewall-cmd --permanent --zone=public --add-port=9095-9096/tcp
$ sudo firewall-cmd --reload
```

теперь переходим на сайт `http://%ip%:9096/` и настраиваем подключение

```
= Language: English

= Catalog API
DB: PostgreSQL
DBName: bacula
Login: bacula
Password: bacula
IP adress: localhost
Port: 5432

= Console API
Bconsole binary file path: /usr/bin/bconsole
Bconsole admin config file path: /opt/bacula/etc/bconsole.conf
Use sudo: yes

= Config API
General configuration
	Directory path for new config files: /opt/bacula/working ($ sudo chown apache /opt/bacula/working)
	Use sudo: yes
Director
	bdirjson binary file path: /usr/bin/bdirjson
	Main Director config file path (usually bacula-dir.conf): /opt/bacula/etc/bacula-dir.conf
Storage Daemon
	bsdjson binary file path: /usr/bin/bsdjson
	Main Storage Daemon config file path (usually bacula-sd.conf): /opt/bacula/etc/bacula-sd.conf
File Daemon/Client
	bfdjson binary file path: /usr/bin/bfdjson
	Main File Daemon config file path (usually bacula-fd.conf): /opt/bacula/etc/bacula-fd.conf
Bconsole
	bbconsjson binary file path: /usr/bin/bbconsjson
	Admin Bconsole config file path (usually bconsole.conf): /opt/bacula/etc/bconsole.conf
```

Далее переходим в сам web-интерфейс `baculum` `http://%ip%:9095/` и так же настраиваем подключение

```
Protocol: http
IP Address/Hostname: localhost
Port: 9096

Use HTTP Basic authentication

API Login: ...
API Password: ...
```
