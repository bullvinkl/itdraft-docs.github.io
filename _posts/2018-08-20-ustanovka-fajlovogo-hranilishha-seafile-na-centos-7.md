---
title: "Установка файлового хранилища SeaFile в Centos 7"
date: "2018-08-20"
categories: 
  - Storage-System
tags: 
  - "apache"
  - "centos"
  - "mysql"
  - "seafile"
image:
  path: /commons/working-on-computer.jpg
  alt: "файловое хранилище SeaFile"
---

> **SeaFile** - файловое хранилище, система с открытым исходным кодом. Аналог Dropbox. На официальном сайте можно скачать клиенты для синхронизации файлов как для десктопных операционных систем (Windows, Linux, Mac), так и для мобильных устройств (Android, iOS)
{: .prompt-tip }

## Установка Seafile

Добавляем репозиторий EPEL и обновляемся

```sh
$ sudo yum install epel-release
$ sudo yum update
```

Устанавливаем необходимый набор софта

```sh
$ sudo yum install nano python-imaging MySQL-python python-simplejson python-setuptools mariadb mariadb-server
```

Отключаем Selinux

```sh
$ sudo setenforce 0
$ sudo nano /etc/sysconfig/selinux
SELINUX=disabled
```

Устанавливаем [Web-сервер Apache]({% post_url 2018-07-27-ustanovka-web-servera-apache-na-centos-7 %}), добавляем его в автозагрузку и запускаем

Добавляем поддержку ssl в Apache и перезапускаем службу

```sh
$ sudo yum install mod_ssl
$ sudo systemctl restart httpd.service
```

Устанавливаем [MySQL-сервер (MariaDB)]({% post_url 2018-07-27-ustanovka-mysql-servera-mariadb-na-centos-7 %}), добавляем его в автозагрузку и запускаем

Подключаемся к базе данных под root-пользователем

```sh
$ sudo mysql -u root -p
```

Создаем 3 базы, пользователя, задаем пароль для пользователя и назначаем ему привилегии

```sh
> create database ccnet_db character set = 'utf8';
> create database seafile_db character set = 'utf8';
> create database seahub_db character set = 'utf8';
> create user 'seafile'@'localhost' identified by 'password';
> GRANT ALL PRIVILEGES ON `ccnet_db`.* to `seafile`@`localhost`;
> GRANT ALL PRIVILEGES ON `seafile_db`.* to `seafile`@`localhost`;
> GRANT ALL PRIVILEGES ON `seahub_db`.* to `seafile`@`localhost`;
> FLUSH PRIVILEGES;
> exit;
```

Создаем каталог, куда будем ставить SeaFile, переходим в него и скачиваем дистрибутив с официального сайта

```sh
$ sudo mkdir -p /home/seafile
$ cd /home/seafile
$ sudo wget https://download.seadrive.org/seafile-server_6.0.9_x86-64.tar.gz
```

Разархивируем дистрибутив, переименовываем и запускаем процесс установки

```sh
$ sudo tar -xzvf seafile-server_6.0.9_x86-64.tar.gz
$ sudo mv seafile-server-6.0.9 seafile-server
$ sudo cd seafile-server/
$ sudo ./setup-seafile-mysql.sh
```

В процессе установке система спросит некоторые вопросы, на которые надо будет ответить

```
server name - seafile
server's ip or domain - 192.168.1.45
default data dirctory - just press Enter
default port - press Enter
Now for the database configuration, choose number 2
For the MySQL configuration:
use deafult host - localhost
default port - 3306
the mysql user - 'seafile'
and the password is 'password'
ccnet database is 'ccnet_db'
seafile database is 'seafile_db'
seahub database is 'seahub_db'

Your seafile server configuration has been finished successfully.
-----------------------------------------------------------------
run seafile server:     ./seafile.sh { start | stop | restart }
run seahub  server:     ./seahub.sh  { start <port> | stop | restart <port> }
-----------------------------------------------------------------
If you are behind a firewall, remember to allow input/output of these tcp ports:
-----------------------------------------------------------------
port of seafile fileserver:   8082
port of seahub:               8000
```

Меняем владельца и группу каталога, куда установлен SeaFile и каталога с временными файлами SeaFile

```sh
$ sudo chown -R apache:apache /home/seafile
$ sudo chown -R apache:apache /tmp/seahub_cache
```

Создаем системный юнит `seafile.service`

```sh
$ sudo nano /etc/systemd/system/seafile.service
[Unit]
Description=Seafile Server
Before=seahub.service
After=network.target mariadb.service

[Service]
Type=oneshot
ExecStart=/home/seafile/seafile-server-6.0.9/seafile.sh start
ExecStop=/home/seafile/seafile-server-6.0.9/seafile.sh stop
RemainAfterExit=yes
User=apache
Group=apache

[Install] WantedBy=multi-user.target
```

Создаем системный юнит `seahub.service`

```sh
$ sudo nano /etc/systemd/system/seahub.service
[Unit]
Description=Seafile Hub
After=network.target seafile.target mariadb.service

[Service]
Type=oneshot
ExecStart=/home/seafile/seafile-server-6.0.9/seahub.sh start
ExecStop=/home/seafile/seafile-server-6.0.9/seahub.sh stop
RemainAfterExit=yes
User=apache
Group=apache

[Install] WantedBy=multi-user.target
```

Перечитываем изменения в системных юнитах и запускаем созданные сервисы

```sh
$ sudo systemctl daemon-reload
$ sudo systemctl start seafile
$ sudo systemctl start seahub
```

## Настройка Apache

Добавляем `vhosts` - несколько сайтов на одном ip-адресе

```sh
$ sudo nano /etc/httpd/conf.d/vhosts.conf
# Загрузка моих vhosts
IncludeOptional vhosts.d/*.conf
```

Создаем каталог, где будут лежать конфигурации `vhosts`

```sh
$ sudo mkdir /etc/httpd/vhosts.d
```

Создаем конфигурационный файл для SeaFile

```sh
$ sudo nano /etc/httpd/vhosts.d/seafile.example.ru.conf
<VirtualHost *:80>
 ServerName seafile.example.ru
 ServerSignature Off
 
 Redirect / https://seafile.example.ru/

</VirtualHost>

<VirtualHost *:443>
 ServerName seafile.example.ru
 ServerSignature Off

 Alias /media  /home/seafile/seafile-server-latest/seahub/media

# Каталог, где лежать ssl-сертификаты надо создать заранее
 SSLEngine on
 SSLCertificateFile /etc/httpd/ssl/example.ru/certificate.crt
 SSLCertificateKeyFile /etc/httpd/ssl/example.ru/private.key

 SSLProxyEngine on
 SSLProxyCheckPeerCN on
 SSLProxyCheckPeerExpire on

# seafile httpserver
 ProxyPass /seafhttp http://127.0.0.1:8082
 ProxyPassReverse /seafhttp http://127.0.0.1:8082
 RewriteRule ^/seafhttp - [QSA,L]

# seahub
 ProxyPass / http://127.0.0.1:8000/
 ProxyPassReverse / http://127.0.0.1:8000/
 RewriteRule ^/(media.*)$ /$1 [QSA,L,PT]
 RewriteCond %{REQUEST_FILENAME} !-f
 RewriteRule ^(.*)$ /seahub.fcgi$1 [QSA,L,E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]

 Timeout 2400
 ProxyTimeout 2400
 ProxyBadHeader Ignore

    ErrorLog /home/seafile/error_ssl.log
    CustomLog /home/seafile/access_ssl.log combined
</VirtualHost>
```

Перезапускаем наши сервисы

```sh
$ sudo systemctl restart httpd
$ sudo systemctl restart seafile
$ sudo systemctl restart seahub
```

Добавляем их в автозагрузку

```sh
$ sudo systemctl enable httpd
$ sudo systemctl enable mariadb
$ sudo systemctl enable seafile
$ sudo systemctl enable seahub
```

## Настройка Firewall

Открываем необходимые порты для SeaFile и перезапускаем службу firewall

```sh
$ sudo firewall-cmd --zone=public --add-port=8000/tcp --permanent
$ sudo firewall-cmd --zone=public --add-port=8082/tcp --permanent
$ sudo firewall-cmd --zone=public --add-port=10001/tcp --permanent
$ sudo firewall-cmd --zone=public --add-port=12001/tcp --permanent
$ sudo firewall-cmd --reload
```

## Дополнительная информация

Если нужно исправить URL, по которому загружается SeaFile, правим один из конфигурационных файлов и перзапускаем SeaFile

```sh
$ sudo nano /home/seafile/conf/ccnet.conf
SERVICE_URL = https://share.example.ru

$ sudo systemctl restart seafile
$ sudo systemctl restart seahub
```
