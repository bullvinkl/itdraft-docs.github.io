---
title: "Установка ITSM-системы CMDBuild в Centos 7"
date: "2020-02-19"
categories: 
  - Asset-Management
tags: 
  - "assetmanager"
  - "centos"
  - "cmdbuild"
  - "openjdk"
  - "postgresql"
  - "tomcat"
image:
  path: /commons/1361114_fab1_2.jpg
  alt: "Установка ITSM-системы CMDBuild"
---

> **CMDBuild** — это программное обеспечение с открытыми исходными кодами, которое управляет конфигурацией базы данных. CMDBuild был спроектирован в соответствии с ITIL "лучше из применяемых на практике способов" для IT услуг управлении, отвечая на критерии ориентированные на процесс.
{: .prompt-tip }

## Используемое программное обеспечение

- CentOS 7.7 1908 Minimal
- PostgreSQL 10.12
- OpenJDK 13.0.2
- Tomcat 9.0.31
- CMDBuild 3.2

## Подготовка

Обновимся и установим репозиторий EPEL и необходимый софт

```sh
$ sudo yum update
$ sudo yum -y install epel-release
$ sudo yum -y install wget nano
```

## Устанавливаем PostgreSQL 10

Добавляем репозиторий

```sh
$ sudo yum -y install https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm
```

Устанавливаем PostgreSQL

```sh
$ sudo yum -y install postgresql10 postgresql10-server postgresql10-contrib postgresql10-libs
```

Инициализируем БД

```sh
$ sudo /usr/pgsql-10/bin/postgresql-10-setup initdb
```

Настраиваем доступ к базе PostgreSQL

```sh
$ sudo nano /var/lib/pgsql/10/data/pg_hba.conf
# TYPE  DATABASE        USER            ADDRESS                 METHOD

# "local" is for Unix domain socket connections only
local   all             all                                     trust
# IPv4 local connections:
host    all             all             127.0.0.1/32            md5
# IPv6 local connections:
host    all             all             ::1/128                 md5
```

Добавляем PostgreSQL в автозагрузку и запускаем сервис

```sh
$ sudo systemctl enable --now postgresql-10
```

Создаем базу / пользователей

```sh
$ sudo su - postgres
-bash-4.2$ psql
postgres=# CREATE DATABASE cmdbuild_db;
postgres=# CREATE USER cmdbuild_user WITH encrypted password 'password';
postgres=# GRANT ALL PRIVILEGES ON DATABASE cmdbuild_db TO cmdbuild_user;
postgres=# DROP DATABASE cmdbuild_db;
postgres=# CREATE USER cmdbuild_superuser WITH encrypted password 'password';
postgres=# ALTER USER cmdbuild_superuser WITH SUPERUSER;
postgres=# \q
-bash-4.2$ exit
```

Базу удалили специально, т.к. в дальнейшем она будет создана установочным скриптом

## Устанавливаем OpenJDK 13 и Tomcat 9

Устанавливаем OpenJDK latest (13.0.2)

```sh
$ sudo yum -y install java-latest-openjdk java-latest-openjdk-devel
$ java --version
```

Создаем системного пользователя Tomcat

```sh
$ sudo useradd -m -U -d /opt/tomcat -s /bin/false tomcat
```

Скачиваем архив Tomcat 9.0.31 и распаковываем его

```sh
$ cd /tmp
$ wget http://apache-mirror.rbc.ru/pub/apache/tomcat/tomcat-9/v9.0.31/bin/apache-tomcat-9.0.31.tar.gz
$ tar -xf apache-tomcat-9.0.31.tar.gz
$ sudo mv apache-tomcat-9.0.31 /opt/tomcat/
```

Создаем сим линк, меняем владельца директорий и делаем bash-скрипты исполняемыми

```sh
$ sudo ln -s /opt/tomcat/apache-tomcat-9.0.31 /opt/tomcat/latest
$ sudo chown -R tomcat:tomcat /opt/tomcat
$ sudo sh -c 'chmod +x /opt/tomcat/latest/bin/*.sh'
```

Создаем Systemd Unit

```sh
$ sudo nano /etc/systemd/system/tomcat.service
[Unit]
Description=Tomcat 9 servlet container
After=network.target

[Service]
Type=forking

User=tomcat
Group=tomcat

Environment="JAVA_HOME=/usr/lib/jvm/jre"
Environment="JAVA_OPTS=-Djava.security.egd=file:///dev/urandom"

Environment="CATALINA_BASE=/opt/tomcat/latest"
Environment="CATALINA_HOME=/opt/tomcat/latest"
Environment="CATALINA_PID=/opt/tomcat/latest/temp/tomcat.pid"
Environment="CATALINA_OPTS=-Xms512M -Xmx1024M -server -XX:+UseParallelGC"

ExecStart=/opt/tomcat/latest/bin/startup.sh
ExecStop=/opt/tomcat/latest/bin/shutdown.sh

[Install]
WantedBy=multi-user.target
```

Добавляем Tomcat в автозагрузку и запускаем сервис

```sh
$ sudo systemctl daemon-reload
$ sudo systemctl enable --now tomcat
$ sudo systemctl status tomcat
```

## Настраиваем Firewall

Открываем порт 8080

```sh
$ sudo firewall-cmd --zone=public --permanent --add-port=8080/tcp
$ sudo firewall-cmd --reload
```

## Настройка доступа к Tomcat Web Management Interface

Редактируем файл `conf/tomcat-users.xml`, устанавливаем логин/пароль

```sh
$ sudo nano /opt/tomcat/latest/conf/tomcat-users.xml
...
   <role rolename="admin-gui"/>
   <role rolename="manager-gui"/>
   <user username="admin" password="admin_password" roles="admin-gui,manager-gui"/>
</tomcat-users>
```

Редактируем файл `manager/META-INF/context.xml`, добавляем IP

```sh
$ sudo nano /opt/tomcat/latest/webapps/manager/META-INF/context.xml
...
<Context antiResourceLocking="false" privileged="true" >
  <Valve className="org.apache.catalina.valves.RemoteAddrValve"
         allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1|10.0.2.2" />
</Context>
```

Редактируем файл `host-manager/META-INF/context.xml`, добавляем IP

```sh
$ sudo nano /opt/tomcat/latest/webapps/host-manager/META-INF/context.xml
...
<Context antiResourceLocking="false" privileged="true" >
  <Valve className="org.apache.catalina.valves.RemoteAddrValve"
         allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1|10.0.2.2" />
</Context>
```

где `10.0.2.2` - разрешенный IP (virtualbox host)

Либо в этих 2-х файлах `context.xml` (manager / host-manager) закоментить строку

```
<!--
<Valve className="org.apache.catalina.valves.RemoteAddrValve" ... />
-->
```

Так же можно удалить приложения, установленные по-умолчанию, но я их оставил

```sh
$ sudo rm -rf /opt/tomcat/latest/webapps/*
```

## Установка CMDBuild 3.2

Скачиваем файл `cmdbuild.war` и копируем его в директорию Tomcat

```sh
$ cd /tmp
$ wget -O cmdbuild-3.2.war https://sourceforge.net/projects/cmdbuild/files/3.2/cmdbuild-3.2.war/download?use_mirror=netcologne#
$ sudo cp /tmp/cmdbuild-3.2.war /opt/tomcat/latest/webapps/cmdbuild.war
$ sudo chown tomcat:tomcat /opt/tomcat/latest/webapps/cmdbuild.war
```

Настраиваем подключение к базе PostgreSQL

```sh
$ sudo mkdir -p /opt/tomcat/latest/conf/cmdbuild
$ sudo nano /opt/tomcat/latest/conf/cmdbuild/database.conf
db.url=jdbc:postgresql://localhost:5432/cmdbuild_db
db.username=cmdbuild_user
db.password=password
db.admin.username=cmdbuild_superuser
db.admin.password=password
```

Меняем владельца

```sh
$ sudo chown -R tomcat:tomcat /opt/tomcat/latest/conf/cmdbuild
```

Скачиваем доп. библиотеки, копируем их в директорию Tomcat, меняем владельца

```sh
$ cd /tmp
$ wget -O cmdbuild-3.2-resources.tar.gz https://sourceforge.net/projects/cmdbuild/files/3.2/cmdbuild-3.2-resources.tar.gz/download#
$ tar -xzf /tmp/cmdbuild-3.2-resources.tar.gz
$ sudo cp /tmp/tomcat-libs/postgresql-*.jar /opt/tomcat/latest/lib/
$ sudo chown -R tomcat:tomcat /opt/tomcat/latest/lib
```

Создаем структуру базы

```sh
$ sudo systemctl stop tomcat
$ sudo chmod +x /opt/tomcat/latest/webapps/cmdbuild/cmdbuild.sh
$ sudo bash /opt/tomcat/latest/webapps/cmdbuild/cmdbuild.sh dbconfig create empty -configfile /opt/tomcat/latest/conf/cmdbuild/database.conf
$ sudo systemctl restart tomcat
```

Если нужно загрузить демо-данные:

```sh
$ sudo bash /opt/tomcat/latest/webapps/cmdbuild/cmdbuild.sh dbconfig create demo -configfile /opt/tomcat/latest/conf/cmdbuild/database.conf
```

предварительно удалив базу

```sh
postgres=# DROP DATABASE cmdbuild_db;
```

Проверяем:

```
http://localhost:8080/cmdbuild/?language=ru_RU  
Пользователь: admin  
Пароль: admin
```
