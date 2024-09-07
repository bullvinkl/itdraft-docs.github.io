---
title: "Установка CalDAV / CardDAV сервиса Davical 1.1.5 + LDAP на CentOS 6.8"
date: "2017-04-21"
categories: 
  - Communication-System
tags: 
  - "caldav"
  - "carddav"
  - "centos"
  - "davical"
  - "postgresql"
image:
  path: /commons/752128_6d9f_2.jpg
  alt: "Установка CalDAV / CardDAV"
---

> **CalDAV** (Calendaring Extensions to WebDAV) - это сетевой протокол, расширяющий WebDAV и позволяющий синхронизировать информацию о планировании времени между клиентами и сервером. Он позволяет пользователям синхронизировать свой календарь с CalDAV-сервером и использовать его на различных устройствах.
> **CardDAV** (vCard Extensions to WebDAV) - это открытый Интернет-стандарт для взаимодействия с информацией в адресных книгах. Он позволяет клиентам CardDAV обращаться к адресным книгам, хранимым на сервере, и синхронизировать контактную информацию между устройствами.
{: .prompt-tip }

Скачиваем davical и awd

```sh
$ wget https://gitlab.com/davical-project/awl/repository/archive.tar.gz?ref=master -O awl.tar.gz
$ wget https://gitlab.com/davical-project/davical/repository/archive.tar.gz?ref=master -O davical.tar.gz
```

Распаковываем

```sh
$ tar -xzf awl.tar.gz 
$ tar -xzf davical.tar.gz
```

Перемещаем

```sh
$ sudo mv awl-master-4c75c662e8605ed54ba4b8e65e4c3a8cc773052f/ /usr/share/awl
$ sudo mv davical-master-8313f765ce89f752af77e0e0a90f3d1f5981b5b5/ /usr/share/davical
```

Меняем права

```sh
$ sudo chmod  755 -R /usr/share/awl/
$ sudo chmod  755 -R /usr/share/davical/
```

Устанавливаем postgresql 9.6

```sh
$ sudo rpm -Uvh https://yum.postgresql.org/9.6/redhat/rhel-6-x86_64/pgdg-redhat96-9.6-3.noarch.rpm
$ sudo yum install postgresql96-server postgresql96 postgresql96-lib
```

Инициализируем базу данных

```sh
$ sudo service postgresql-9.4 initdb
```

Запускаем сервис и добавляем в автозагрузку

```sh
$ sudo service postgresql-9.4 start
$ sudo chkconfig postgresql-9.4 on
```

Настраитваем PostgreSQL 9.6
меняем `listen_addresses = 'localhost'` на `listen_addresses = '*'`

```sh
$ sudo nano /var/lib/pgsql/9.6/data/postgresql.conf
```

Разрешаем доступ без авторизации (не безопасно)

```sh
$ sudo nano /var/lib/pgsql/9.6/data/pg_hba.conf
...
# "local" is for Unix domain socket connections only
local   all             all                                     trust
host    all             all             127.0.0.1/32            trust
host    all             all             192.168.1.0/24          trust
local    davical         davical_app                             trust
local    davical         davical_dba                             trust
```

Открывает порт в iptables

```sh
$ sudo nano /etc/sysconfig/iptables
```

Добавляем строчку:

```
-A INPUT -m state --state NEW -m tcp -p tcp --dport 5432 -j ACCEPT
```

Перезагружаем postgresql и файерволл

```sh
$ sudo service postgresql-9.6 restart
$ sudo service iptables restart
```

Продолжаем установку `davical`, инициализируем базу

```sh
$ sudo su - postgres -c /usr/share/davical/dba/create-database.sh
```

Если база инициировалась успешно, высветится сообщение, где будет указан логин и пароль

```
Supported locales updated.
Updated view: dav_principal.sql applied.
CalDAV functions updated.
RRULE functions updated.
Database permissions updated.
NOTE
====
*  The password for the 'admin' user has been set to 'password'

Thanks for trying DAViCal! Check the configuration in /etc/davical/config.php.
For help, look at our website and wiki, or visit #davical on irc.oftc.net.
```

Если не получилось создать базу, разбираемся в чем проблема, удаляем не до конца созданную базу

```sh
$ sudo su postgres -c "dropdb davical"
```

У меня проблема возникла из-за не правильной настроки `postgresql`, точнее из-за строчки в файле `pg_hba.conf`

```
local   all             all                                     trust
```

Конфигурационный файл Apache (/etc/httpd/virtual/davical.local)

```
  DocumentRoot /usr/share/davical/htdocs
  DirectoryIndex index.php index.html
  ServerName calendar.domainnamehere.com
  ServerAlias davical.domainnamehere.com
  Alias /images/ /usr/share/davical/htdocs/images/
  
     AllowOverride None
     Order allow,deny
     Allow from all
  

  php_value include_path /usr/share/awl/inc
  php_value magic_quotes_gpc 0
  php_value register_globals 0
  php_value error_reporting "E_ALL & ~E_NOTICE"
  php_value default_charset "utf-8"

  ErrorLog "logs/calendar.domainnamehere.com-error_log"
  CustomLog "logs/calendar.domainnamehere.com-access_log" common

```

Конфигурационный файл davical (/usr/share/davical/config/config.php)

```
<?php
  $c->domain_name = "davical.local";
  $c->sysabbr     = 'davical';
  $c->admin_email = 'admin@example.net';
  $c->system_name = "Example DAViCal Server";
  $c->pg_connect[] = 'dbname=davical user=davical_app';
  $c->default_locale = "ru_RU";
```

Конфигурационный файл davical+ldap (/usr/share/davical/config/config.php)

```
<?php
$c->pg_connect[] = "dbname=davical user=davical_app";
$c->system_name = "DAViCal CalDAV Server";
$c->readonly_webdav_collections = false;
$c->admin_email ='admin@example.net';
$c->collections_always_exist = true;

$c->template_usr = array( 'active' => true,
                       'locale' => 'ru_RU',
                       'date_format_type' => 'E',
                       'email_ok' => date('Y-m-d')
                     );

$c->schedule_private_key = 'PRIVATE-KEY-BASE-64-DATA';
$c->external_refresh = 60;
$c->support_obsolete_free_busy_property = true;


$c->authenticate_hook['call'] = 'LDAP_check';
$c->authenticate_hook['config'] = array(
    'host'              => '192.168.1.2',
    'port'              => '389',
    'bindDN'            => 'admin@DOMAIN',
    'passDN'            => 'password',
    'baseDNUsers'       => 'dc=domain,dc=local',
//    'filterUsers'       => 'objectClass=InetOrgPerson',
    'protocolVersion'   => 3,
    'optReferrals'      => 0,
    'filterUsers'       => '(&(objectcategory=person)(objectclass=user)(givenname=*))',
    'mapping_field'     => array('username' => 'sAMAccountName',
                                 'updated'  => 'whenChanged',
                                 'fullname' => 'cn' ,
                                 'email'    => 'mail'),
      'default_value' => array("date_format_type" => "E","locale" => "ru_RU"),
//    'default_value'     => array("date_format_type" => "E","locale" => "en_NZ"),
    'format_updated'    => array('Y' => array(0,4),'m' => array(4,2),'d'=> array(6,2),'H' => array(8,2),'M'=>array(10,2),'S' => array(12,2))
//    'scope' => 'subtree',
    );
//
//  /* If there is some user you do not want to sync from LDAP, put their username in this list */
  $c->do_not_sync_from_ldap = array( 'admin' => true );
include('drivers_ldap.php');

 $c->default_locale = "ru_RU";
 $c->domain_name = "davical.local";
 $c->local_tzid = 'Europe/Moscow';
 $c->allow_get_email_visibility = true;
```
