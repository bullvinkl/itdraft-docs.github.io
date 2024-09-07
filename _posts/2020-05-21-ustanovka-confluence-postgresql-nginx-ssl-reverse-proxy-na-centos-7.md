---
title: "Установка Confluence + PostgreSQL + Nginx SSL reverse-proxy на Centos 7"
date: "2020-05-21"
categories: 
  - Project-Management
  - Database-System
  - Web
tags: 
  - "atlassian"
  - "confluence"
  - "nginx"
  - "postgresql"
  - "reverse-proxy"
  - "ssl"
image:
  path: /commons/1292838_f55d.jpg
  alt: ""
---

> **Confluence** — тиражируемая вики-система для внутреннего использования организациями с целью создания единой базы знаний. Написана на Java. Разрабатывается австралийской компанией Atlassian, является одним из двух её основных продуктов.
{: .prompt-tip }

## Установка PostgreSQL 12

Добавляем репозиторий PostgreSQL

```sh
$ sudo yum -y install https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm
```

Устанавливаем необходимые пакеты

```sh
$ sudo yum -y install epel-release yum-utils
$ sudo yum-config-manager --enable pgdg12
$ sudo yum -y install postgresql12-server postgresql12
```

После установки требуется инициализация базы данных, прежде чем можно будет запустить службу

```sh
$ sudo /usr/pgsql-12/bin/postgresql-12-setup initdb
```

Запускаем сервис PostgreSQL и проверяем статус

```sh
$ sudo systemctl enable --now postgresql-12
$ systemctl status postgresql-12
```

Редактируем настройки PostgreSQL, открываем доступ для Confluence

```sh
$ sudo nano /var/lib/pgsql/12/data/pg_hba.conf
[...]
# IPv4 local connections:
#host    all             all             127.0.0.1/32            ident
host    confluence      confluenceuser  127.0.0.1/32            md5
```

Перезапускаем PostgreSQL

```sh
$ sudo systemctl restart postgresql-12
```

Создаем пользователя и базу

```sh
$ sudo su - postgres
$ psql
postgres=# CREATE ROLE confluenceuser WITH LOGIN PASSWORD 'password' VALID UNTIL 'infinity';
CREATE ROLE
postgres=# CREATE DATABASE confluence WITH ENCODING='UTF8' OWNER=confluenceuser CONNECTION LIMIT=-1;
CREATE DATABASE
postgres-# \q
$ exit
```

## Установка Confluence

Создаем пользователя, от которого будет работать Confluence

```sh
$ sudo useradd -m -U -r -d /opt/atlassian confluence
```

Задаем пароль пользователю

```sh
$ sudo passwd confluence
New password:
Retype new password:
```

Добавляем пользователя `confluence` в группу `wheel`, что бы у него появились права суперпользователя (`sudo`)

```sh
$ sudo usermod -aG wheel confluence
```

Переключаемся на пользователя `confluence`, переходим в домашний каталог. Все дальнейшие операции будут выполняться из-под этого пользователя

```sh
$ sudo su confluence
$ cd
```

Скачиваем файл дистрибутива `confluence 7.5.0`, и делаем файл исполняемым

```sh
$ wget https://product-downloads.atlassian.com/software/confluence/downloads/atlassian-confluence-7.5.0-x64.bin
$ chmod a+x atlassian-confluence-7.5.0-x64.bin
```

Запускаем установку Confluence

```sh
$ sudo ./atlassian-confluence-7.5.0-x64.bin
```

В процессе установки надо будет выбирать действия

```sh
This will install Confluence 7.5.0 on your computer.
OK [o, Enter], Cancel [c]
o
Click Next to continue, or Cancel to exit Setup.

Choose the appropriate installation or upgrade option.
Please choose one of the following:
Express Install (uses default settings) [1],
Custom Install (recommended for advanced users) [2, Enter],
Upgrade an existing Confluence installation [3]
2

Select the folder where you would like Confluence 7.5.0 to be installed, then click Next.
Where should Confluence 7.5.0 be installed?
[/opt/atlassian/confluence]


Default location for Confluence data
[/var/atlassian/application-data/confluence]


Configure which ports Confluence will use.
Confluence requires two TCP ports that are not being used by any other applications on this machine. The HTTP port is where you will access Confluence through your browser. The Control port is used to Startup and
Shutdown Confluence.
Use default ports (HTTP: 8090, Control: 8000) - Recommended [1, Enter], Set custom value for HTTP and Control ports [2]
1

Confluence can be run in the background.
You may choose to run Confluence as a service, which means it will start automatically whenever the computer restarts.
Install Confluence as Service?
Yes [y, Enter], No [n]
y

Extracting files ...

Please wait a few moments while we configure Confluence.

Installation of Confluence 7.5.0 is complete
Start Confluence now?
Yes [y, Enter], No [n]
y

Please wait a few moments while Confluence starts up.
Launching Confluence ...

Installation of Confluence 7.5.0 is complete
Your installation of Confluence 7.5.0 is now ready and can be accessed via
your browser.
Confluence 7.5.0 can be accessed at http://localhost:8090
Finishing installation ...
```

Настраиваем Firewall, открываем порт 8090/tcp

```sh
$ sudo firewall-cmd --permanent --add-port=8090/tcp
$ sudo firewall-cmd --reload
```

Проверяем, запустился ли Confluence

```sh
$ netstat -nltup | grep 8090
```

Если записи с номером порта нет, запускаем Confluence вручную

```sh
$ /etc/init.d/confluence start
либо
$ sudo /opt/atlassian/confluence/bin/catalina.sh start
```

Переходим на сайт `http://localhost:8090` и продолжаем установку


![](/assets/img/posts/2020/05/21/confluece1.png){: w="300" }
_Промышленная установка_

![](/assets/img/posts/2020/05/21/confluece2.1-1.png){: w="300" }
_Триальная лицензия_


На сайте [atlassian](https://my.atlassian.com/license/evaluation) генерим триальную лицензию по идентификатору сервера

![](/assets/img/posts/2020/05/21/confluece2-1.png){: w="300" }
_Триальная лицензия_

![](/assets/img/posts/2020/05/21/confluece3.png){: w="300" }
_Моя база данных_

![](/assets/img/posts/2020/05/21/confluece4.png){: w="300" }
_Настройка базы данных_

Вводим данные по подключению к PostgreSQL и жмем кнопку "Проверить соединение"

```
	Тип базы данных: PostgreSQL
	Тип установки: Простой
	Имя хоста: localhost
	Порт: 5432
	Название базы данных: confluence
	Имя пользователя: confluenceuser
	Пароль: password
```

![](/assets/img/posts/2020/05/21/confluece5.png){: w="300" }
_Пример сайта_

Для первого раза рекомендую установить пример сайта. Вы всегда сможете удалить это тестовое пространство

![](/assets/img/posts/2020/05/21/confluece6.png){: w="300" }
_Настройка управления пользователями - Управление пользователями и группами в Confluence_

![](/assets/img/posts/2020/05/21/confluece7.1.png){: w="300" }
_Настройка учетной записи системного администратора_

![](/assets/img/posts/2020/05/21/confluece8.png){: w="300" }
_Установка завершена_

## Настройка Nginx в качестве reverse-proxy

Добавляем репозиторий Nginx

```sh
$ sudo nano /etc/yum.repos.d/nginx.repo
[nginx-stable]
name=nginx stable repo
baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=1
enabled=1
gpgkey=https://nginx.org/keys/nginx_signing.key
 
[nginx-mainline]
name=nginx mainline repo
baseurl=http://nginx.org/packages/mainline/centos/$releasever/$basearch/
gpgcheck=1
enabled=0
gpgkey=https://nginx.org/keys/nginx_signing.key
```

Устанавливаем Nginx, добавляем службу в автозагрузку и запускаем его

```sh
$ sudo yum install -y nginx
$ sudo systemctl enable --now nginx
```

Создаем каталог, где будет лежать самоподписанный ssl сертификат

```sh
$ sudo mkdir /etc/nginx/ssl
$ sudo chmod 700 /etc/nginx/ssl
```

Создаем самоподписанный сертификат и ключ

```sh
$ sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/nginx/ssl/nginx.key -out /etc/nginx/ssl/nginx.crt
Country Name (2 letter code) [XX]: RU
State or Province Name (full name) []: Moscow
Locality Name (eg, city) [Default City]: Moscow
Organization Name (eg, company) [Default Company Ltd]: Company
Organizational Unit Name (eg, section) []: IT
Common Name (eg, your name or your server's hostname) []: localhost
Email Address []: admin@itdraft.ru

$ sudo openssl dhparam -out /etc/nginx/ssl/dhparam.pem 2048
```

Отредактируем файл конфигурации Nginx

```sh
$ sudo nano /etc/nginx/conf.d/default.conf
server {
 server_name localhost;
 
 listen 443 default ssl;
 ssl_certificate /etc/nginx/ssl/nginx.crt;
 ssl_certificate_key /etc/nginx/ssl/nginx.key;
 ssl_dhparam /etc/nginx/ssl/dhparam.pem;
 
 ssl_session_timeout 5m;
 
 ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
 ssl_ciphers 'ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA:ECDHE-RSA-AES256-SHA:ECDHE-ECDSA-DES-CBC3-SHA:ECDHE-RSA-DES-CBC3-SHA:EDH-RSA-DES-CBC3-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:DES-CBC3-SHA:!DSS';
 ssl_prefer_server_ciphers on;
 
 location / {
 client_max_body_size 100m;
 proxy_set_header X-Forwarded-Host $host;
 proxy_set_header X-Forwarded-Server $host;
 proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
 proxy_pass http://localhost:8090;
 }
 location /synchrony {
 proxy_set_header X-Forwarded-Host $host;
 proxy_set_header X-Forwarded-Server $host;
 proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
 proxy_pass http://localhost:8091/synchrony;
 proxy_http_version 1.1;
 proxy_set_header Upgrade $http_upgrade;
 proxy_set_header Connection "Upgrade";
 }
}
 
server {
 listen 80;
 server_name localhost;
 return 301 https://$server_name$request_uri;
}
```

Хост `localhost` в строке `server_name` можно заменить на любое доменное имя. На тестовой машине я обычно использую localhost.

Проверим конфиг и перезапускаем Nginx

```sh
$ sudo nginx -t
$ sudo systemctl restart nginx
```

Теперь необходимо сделать настройки со стороны Confluence, правим настройки tomcat

```sh
$ sudo nano /opt/atlassian/confluence/conf/server.xml
```

Закомментируем строку:

```
<!--
< Connector port="8090" connectionTimeout="20000" redirectPort="8443"
maxThreads="48" minSpareThreads="10"
enableLookups="false" acceptCount="10" debug="0" URIEncoding="UTF-8"
protocol="org.apache.coyote.http11.Http11NioProtocol"/ >
-->
```

Раскомментируем и подправим строку ниже:

```
< Connector port="8090" connectionTimeout="20000" redirectPort="8443"
maxThreads="48" minSpareThreads="10"
enableLookups="false" acceptCount="10" debug="0" URIEncoding="UTF-8"
protocol="org.apache.coyote.http11.Http11NioProtocol"
scheme="https" proxyName="localhost" proxyPort="443"/>
```

Если вы не будите использовать ssl, то последняя строка будет выглядеть:

```
scheme="http" proxyName="localhost" proxyPort="80"/>
```

`localhost` так же можно заменить на ваш хост

Перезапускаем Confluence

```sh
$ sudo /etc/init.d/confluence restart
```

## Настраиваем Firewall

Т.к. ранее мы открывали порт 8090, закрываем его

```sh
$ sudo firewall-cmd --permanent --remove-port=8090/tcp
```

Открываем порты 80,443

```sh
$ sudo firewall-cmd --permanent --add-service=http
$ sudo firewall-cmd --permanent --add-service=https
$ sudo firewall-cmd --reload
```

## Настраиваем SELinux

```sh
$ sudo cat /var/log/audit/audit.log | grep nginx | grep denied | audit2allow -M mynginx
$ sudo semodule -i mynginx.pp
$ sudo setsebool httpd_can_network_connect on
$ sudo setsebool httpd_can_network_connect on -P
```

## Завершение настройки Confluence

Обновляем базовый URL в настройках Confluence

```
Администрирование (справа вверху шестерёнка) -> Основные настройки -> Настройки сайта -> Базовый адрес сервера (https://localhost/admin/editgeneralconfig.action)
http://localhost:8090 -> https://localhost
```

## Confluence как системный сервис в Linux

Создаем Systemd Unit `confluence.service`

```sh
$ sudo nano /lib/systemd/system/confluence.service
[Unit]
Description=Confluence
After=network.target

[Service]
Type=forking
User=confluence
PIDFile=/opt/atlassian/confluence/work/catalina.pid
ExecStart=/opt/atlassian/confluence/bin/start-confluence.sh
ExecStop=/opt/atlassian/confluence/bin/stop-confluence.sh
TimeoutSec=200
LimitNOFILE=4096
LimitNPROC=4096

[Install]
WantedBy=multi-user.target
```

Меняем права на файл

```sh
$ sudo chmod 664 /lib/systemd/system/confluence.service
```

После создания Systemd Unit, необходимо перезагрузить процесс самого systemd, для подхвата изменений. Затем запускаем сервис и добавляем его в автозагрузку. Проверяем статус

```sh
$ sudo systemctl daemon-reload
$ sudo systemctl enable --now confluence
$ sudo systemctl status confluence
```

## Ошибка во время уставновки и ее решение

Во время установки появилась ошибка

> Error: dedicated user confluence
{: .prompt-danger }

Решение: изменить пользователя, от которого запускается Confluence с помощью скрипта start-confluence.sh

```sh
$ sudo nano /opt/atlassian/confluence/bin/user.sh
# START INSTALLER MAGIC ! DO NOT EDIT !
CONF_USER="confluence" # user created by installer
# END INSTALLER MAGIC ! DO NOT EDIT !

export CONF_USER
```
