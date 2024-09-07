---
title: "Установка Apache Tomcat 9 и GeoServer на Сentos 7"
date: "2018-12-03"
categories: 
  - Linux
tags: 
  - "centos"
  - "geoserver"
  - "tomcat"
image:
  path: /commons/1404112_d95b_2.jpg
  alt: "Установка Apache Tomcat 9 и GeoServer на Сentos"
---

> **Apache Tomcat** - это веб-сервер и сервер сервлетов, разрабатываемый Apache Software Foundation. Он является открытым и бесплатным программным обеспечением, предназначенным для запуска веб-приложений на платформе Java.
> **GeoServer** — программное обеспечение с открытым исходным кодом, написанное на Java, предоставляющее возможность администрирования и публикации геоданных на сервере.
{: .prompt-tip }

## Подготовительный этап

Обновляем операционную систему, добавляем репозиторий EPEL

```sh
$ sudo yum update
$ sudo yum install epel-release
```

Устанавливаем необходимый софт

```sh
$ sudo yum install htop mc nano wget zip unzip
```

Устанавливаем Java 8. Пакет доступен в официальном репозитории

```sh
$ sudo yum install java-1.8.0-openjdk.x86_64 java-1.8.0-openjdk-devel.x86_64
```

После завершения установки можно проверить установленную версию, используя следующую команду

```sh
$ java -version
openjdk version "1.8.0_191"
OpenJDK Runtime Environment (build 1.8.0_191-b12)
OpenJDK 64-Bit Server VM (build 25.191-b12, mixed mode)
```

## Установка Apache Tomcat 9

С официального сайта Apache Tomcat скачиваем релиз программного обеспечения и распаковываем его

```sh
$ cd ~
$ wget http://apache-mirror.rbc.ru/pub/apache/tomcat/tomcat-9/v9.0.13/bin/apache-tomcat-9.0.13.zip
$ sudo unzip apache-tomcat-9.0.13.zip -d /opt
```

После распаковки был создан каталог с именем `apache-tomcat-9.0.13`. Переименуем его

```sh
$ sudo mv /opt/apache-tomcat-9.0.13 /opt/tomcat
```

Выполним следующую команду, чтобы установить переменную среды `CATALINA_HOME`

```sh
$ echo "export CATALINA_HOME='/opt/tomcat/'" >> ~/.bashrc
$ source ~/.bashrc
```

Не рекомендуется запускать `Apache Tomcat` от пользователя `root`, поэтому мы создадим нового пользователя, который будет запускать сервер Tomcat. Так же изменим права доступа на все файлы каталога `/opt/tomcat/`

```sh
$ sudo useradd -r tomcat --shell /bin/false
$ sudo chown -R tomcat:tomcat /opt/tomcat/
```

Создадим systemd unit, для запуска сервиса `tomcat`, со следующим содержимым

```sh
$ sudo nano /etc/systemd/system/tomcat.service
[Unit]
Description=Apache Tomcat 9
After=syslog.target network.target

[Service]
User=tomcat
Group=tomcat
Type=forking
Environment=CATALINA_PID=/opt/tomcat/tomcat.pid
Environment=CATALINA_HOME=/opt/tomcat
Environment=CATALINA_BASE=/opt/tomcat
ExecStart=/opt/tomcat/bin/startup.sh
ExecStop=/opt/tomcat/bin/shutdown.sh
Restart=on-failure

[Install] 
WantedBy=multi-user.target
```

Сохраняем файл и обновим информацию о юнит-файлах

```sh
$ sudo systemctl daemon-reload
```

## Настройка Apache Tomcat 9

Сделаем исполняемые скрипты запуска службы Tomcat, иначе он не запустится

```sh
$ sudo chmod a+x /opt/tomcat/bin/startup.sh
$ sudo chmod a+x /opt/tomcat/bin/catalina.sh
```

Запускаем Tomcat Apache и добавляем его в автозагрузку

```sh
$ sudo systemctl start tomcat
$ sudo systemctl enable tomcat
```

Откроем порт 8080 в фаерволле, чтобы можно было подключиться к сервису

```sh
$ sudo firewall-cmd --permanent --zone=public --add-port=8080/tcp
$ sudo firewall-cmd --reload
```

## Добавление пользователей

Для того, чтобы открыть доступ к Tomcat Manager, необходимо отредактировать файл `tomcat-users.xml`, добавив в него следующие строки

```sh
$ sudo nano /opt/tomcat/conf/tomcat-users.xml
<role rolename="admin"/>
<role rolename="admin-gui"/> 
<role rolename="admin-script"/>
<role rolename="manager"/>
<role rolename="manager-gui"/>
<!-- <role rolename="manager-script"/> -->
<!-- <role rolename="manager-jmx"/> -->
<!-- <role rolename="manager-status"/> -->
<user name="admin" password="password" roles="admin,manager,admin-gui,admin-script,manager-gui,manager-script,manager-jmx,manager-status" />
</tomcat-users>
```

Не забываем поменять пароль на более защищенный

По умолчанию Tomcat Manager доступен только из браузера, работающего на том же компьютере, что и Tomcat. Если вы хотите удалить это ограничение, вам нужно отредактировать файл context.xml и закомментировать или удалить следующую строку:

```sh
$ sudo nano /opt/tomcat/webapps/manager/META-INF/content.xml
<!--
<Valve className="org.apache.catalina.valves.RemoteAddrValve"
allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1" />
-->
```

Перезапускаем Tomcat

```sh
$ sudo systemctl restart tomcat
```

Теперь что бы попасть на наш серевер с Tomcat, необходимо в браузере набрать: `http://IP_address:8080/manager/html`

## Установка GeoServer

Для установки GeoServer необходимо скачать его с официального сайта и распаковать

```sh
$ cd /home
$ wget http://sourceforge.net/projects/geoserver/files/GeoServer/2.14.1/geoserver-2.14.1-war.zip
$ sudo unzip geoserver-2.14.1-war.zip -d /opt/geoserver
```

Перенесем необходимый файл `geoserver.war` в каталог `webapps`

```sh
$ sudo mv /opt/geoserver/geoserver.war /opt/tomcat/webapps/geoserver.war
```

Что бы попасть на наш GeoServer, необходимо в браузере набрать:  
`http://IP_address:8080/geoserver/web/`

Данные для авторизации по-умолчанию:

```
login: admin
pass: geoserver
```

Если у вас уже стоял GeoServer, но вы забыли `login/password` для доступа в админку, файл с паролями располагается тут:  
`/data/security/usergroup/default/users.xml`

Для изменения пароля, надо заменить строку с зашифрованным паролем:

```
<user enabled="true" name="admin"
password="digest1:D9miXH/hVgfxZJscMbfXtbtliG0WOxhLfsznyWfG38X2pda2JOSV4POi55PQI4tw"/>
```

на

```
<user enabled="true" name="admin" password="plain:PASSWORD"/>
```
