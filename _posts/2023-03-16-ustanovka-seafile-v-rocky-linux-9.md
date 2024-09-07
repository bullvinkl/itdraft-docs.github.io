---
title: "Установка Seafile в Rocky Linux 9"
date: "2023-03-16"
categories: 
  - Storage-System
tags: 
  - "linux"
  - "mysql"
  - "nginx"
  - "rocky-linux"
  - "seafile"
  - "ldap"
image:
  path: /commons/stress-min-1.png
  alt: "Установка Seafile в Rocky Linux 9"
---

> **Seafile** – это облачное хранилище файлов с открытым исходным кодом, аналог Dropbox. Но в отличии от Dropbox, файлы хранятся вашем личном сервере. Файлы могут быть синхронизированы с персональными компьютерами и мобильными устройствами через приложения. Так же функционал Seafile позволяет предоставлять доступ к файлам как внешним пользователям, так и внутренним (другим зарегистрированным пользователям вашего хранилища).
{: .prompt-tip }

- Список статей из категории [Seafile](/tags/seafile/)

## Подготовка

Устанавливаем репозиторий Epel и необходимый софт

```sh
$ sudo dnf -y install epel-release
$ sudo dnf -y install python3 python3-setuptools python3-pip python3-ldap python3-devel java-1.8.0-openjdk \
    curl tar nano pwgen poppler-utils libreoffice-headless libreoffice-pyuno libffi-devel gcc gcc-c++
```

Без этих библиотек не установится mysqlclient для python

```sh
$ sudo dnf -y install mariadb-connector-c-devel mariadb-connector-c
```

Устанавливаем утилиты python

```sh
$ sudo pip3 install --upgrade pip
$ sudo pip3 install --upgrade Pillow
$ sudo pip3 install pylibmc captcha jinja2 sqlalchemy psd-tools django-pylibmc django-simple-captcha python3-ldap
$ sudo pip3 install django==3.2.* sqlalchemy==1.4.3 pycryptodome==3.12.0 cffi==1.14.0
$ sudo pip3 install mysqlclient
```

## Установка Memcache

Устанавливаем Memcache

```sh
$ sudo dnf -y install memcached
```

Запускаем службу memcached и добавляем ее в автозагрузку

```sh
$ sudo systemctl enable --now memcached
$ sudo systemctl status memcached
```

## Установка и настройка NGINX

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

Устанавливаем Nginx

```sh
$ sudo dnf -y install nginx
```

Отключаем дефолтный конфиг

```sh
$ sudo mv /etc/nginx/conf.d/default.conf /etc/nginx/conf.d/default.conf.disabled
```

Отредактируем конфиг `nginx.conf`

```sh
$ sudo nano /etc/nginx/nginx.conf
...
events {
    worker_connections 1024;
    multi_accept on;
    use epoll;
}

http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile on;
    tcp_nopush on;
    server_tokens off;
    server_names_hash_bucket_size 128;
    client_max_body_size 50M;
    tcp_nodelay on;
    client_body_timeout 12;
    client_header_timeout 12;
    keepalive_timeout 15;
    send_timeout 10;
    gzip off;

    include /etc/nginx/conf.d/*.conf;
}
```

Создадим конфиг `seafile.conf`

```sh
$ sudo nano /etc/nginx/conf.d/seafile.conf

log_format seafileformat '$http_x_forwarded_for $remote_addr [$time_local] "$request" $status $body_bytes_sent "$http_referer" "$http_user_agent" $upstream_response_time';

server {
    listen 80;
    server_name seafile.itdraft.ru;

    proxy_set_header X-Forwarded-For $remote_addr;

    location / {
         proxy_pass         http://127.0.0.1:8000;
         proxy_set_header   Host $host;
         proxy_set_header   X-Real-IP $remote_addr;
         proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
         proxy_set_header   X-Forwarded-Host $server_name;
         proxy_set_header   X-Forwarded-Proto $scheme;
         proxy_read_timeout  1200s;

         # used for view/edit office file via Office Online Server
         client_max_body_size 0;

         access_log      /var/log/nginx/seahub.access.log seafileformat;
         error_log       /var/log/nginx/seahub.error.log;
    }
        
    location /seafhttp {
        rewrite ^/seafhttp(.*)$ $1 break;
        proxy_pass http://127.0.0.1:8082;
        client_max_body_size 0;
        proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_connect_timeout  36000s;
        proxy_read_timeout  36000s;

        access_log      /var/log/nginx/seafhttp.access.log seafileformat;
        error_log       /var/log/nginx/seafhttp.error.log;
    }
    location /media {
        root /opt/seafile/seafile-server-latest/seahub;
    }
    location /seafdav {
        proxy_pass         http://127.0.0.1:8080/seafdav;
        proxy_set_header   Host $host;
        proxy_set_header   X-Real-IP $remote_addr;
        proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header   X-Forwarded-Host $server_name;
        proxy_set_header   X-Forwarded-Proto $scheme;
        proxy_read_timeout  1200s;

        client_max_body_size 0;

        access_log      /var/log/nginx/seafdav.access.log seafileformat;
        error_log       /var/log/nginx/seafdav.error.log;
    }
}
```

Проверяем на ошибки

```sh
$ sudo nginx -t
```

Запускаем Nginx и добавляем его в автозагрузку

```sh
$ sudo systemctl enable --now nginx
```

## Настройка SeLinux и Firewall

Отключаем SeLinux. В RHEL9 изменилась процедура отключения SeLinux

```sh
$ sudo setenforce 0
$ sudo grubby --update-kernel ALL --args selinux=0
```

Открываем порты 80,443

```sh
$ sudo firewall-cmd --zone=public --add-service={http,https} --permanent
$ sudo firewall-cmd --reload
```

## Установка MySQL-сервера MariaDB

Устанавливаем необходимые пакеты

```sh
$ sudo dnf -y install mariadb mariadb-server
```

Запускаем сервис mariadb и добавляем его в автозагрузку

```sh
$ sudo systemctl enable --now mariadb
```

Запускаем скрипт инициализации БД, устанавливаем mysql root-пароль, отвечаем на вопросы

```sh
$ sudo mysql_secure_installation
...
Switch to unix_socket authentication [Y/n]
...
Change the root password? [Y/n]
New password: myrootpass
Re-enter new password: myrootpass
...
Remove anonymous users? [Y/n]
...
Disallow root login remotely? [Y/n]
...
Remove test database and access to it? [Y/n]
...
Reload privilege tables now? [Y/n]
...
Thanks for using MariaDB!
```

Пробуем подключиться

```sh
$ mysql -u root -p
```

## Установка Seafile 9

Создаем пользователя seafile

```sh
$ sudo useradd -m -U -r -d /opt/seafile seafile
$ sudo chmod 750 /opt/seafile
```

Добавляем пользователя nginx в группу seafile

```sh
$ sudo usermod -aG seafile nginx
```

Переключаемся на пользователя seafile

```sh
$ sudo su - seafile
```

Скачиваем архив seafile-server\_9.0.10 (на момент написания статьи это была финальная версия) и распаковываем его

```sh
$ curl -OL https://s3.eu-central-1.amazonaws.com/download.seadrive.org/seafile-server_9.0.10_x86-64.tar.gz
$ tar xzf seafile-server_*
```

Запускаем установку seafile, отвечаем на вопросы системы

```sh
$ ./seafile-server-9.0.10/setup-seafile-mysql.sh

Press ENTER to continue

[ server name ] seafile
[ This server's ip or domain ] seafile.itdraft.ru
[ default "8082" ] 

Please choose a way to initialize seafile databases:
[1] Create new ccnet/seafile/seahub databases
[2] Use existing ccnet/seafile/seahub databases
[ 1 or 2 ] 1

What is the host of mysql server?
[ default "localhost" ] 

What is the port of mysql server?
[ default "3306" ] 

What is the password of the mysql root user?
[ root password ] myrootpass

verifying password of user root ...  done

Enter the name for mysql user of seafile. It would be created if not exists.
[ default "seafile" ] 

Enter the password for mysql user "seafile":
(тут лучше использовать пароль без спец.символов @#$%^)
[ password for seafile ] seafilepass

Enter the database name for ccnet-server:
[ default "ccnet-db" ] 

Enter the database name for seafile-server:
[ default "seafile-db" ] 

Enter the database name for seahub:
[ default "seahub-db" ] 

This is your configuration
    server name:            seafile
    server ip/domain:       seafile.itdraft.ru

    seafile data dir:       /opt/seafile/seafile-data
    fileserver port:        8082

    database:               create new
    ccnet database:         ccnet-db
    seafile database:       seafile-db
    seahub database:        seahub-db
    database user:          seafile

[...]
-----------------------------------------------------------------
Your seafile server configuration has been finished successfully.
-----------------------------------------------------------------

run seafile server:     ./seafile.sh { start | stop | restart }
run seahub  server:     ./seahub.sh  { start <port> | stop | restart <port> }

-----------------------------------------------------------------
If you are behind a firewall, remember to allow input/output of these tcp ports:
-----------------------------------------------------------------

port of seafile fileserver:   8082
port of seahub:               8000

When problems occur, Refer to

        https://download.seafile.com/published/seafile-manual/home.md

for information.
```

Переключаемся на основного пользователя (правами sudo)

```sh
$ exit
```

Создаем системный юнит `seafile.service`

```sh
$ sudo nano /etc/systemd/system/seafile.service
[Unit]
Description=Seafile Server
After=network.target remote-fs.target mysqld.service

[Service]
ExecStart=/opt/seafile/seafile-server-latest/seafile.sh start
ExecStop=/opt/seafile/seafile-server-latest/seafile.sh stop
User=seafile
Group=seafile
LimitNOFILE=infinity
Type=oneshot
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
```

Создаем системный юнит `seahub.service`

```sh
$ sudo nano /etc/systemd/system/seahub.service
[Unit]
Description=Seafile Seahub
After=network.target seafile.service

[Service]
ExecStart=/opt/seafile/seafile-server-latest/seahub.sh start
ExecStop=/opt/seafile/seafile-server-latest/seahub.sh stop
User=seafile
Group=seafile
Type=oneshot
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
```

Перечитываем системные юниты

```sh
$ sudo systemctl daemon-reload
```

## Настройка Seafile

Отредактируем `seafdav.conf`, для активации webdav

```sh
$ sudo nano /opt/seafile/conf/seafdav.conf
[WEBDAV]
enabled = true
port = 8080
#fastcgi = true
share_name = /seafdav
```

Проверяем настройки `ccnet.conf`

```sh
$ sudo nano /opt/seafile/conf/ccnet.conf
[General]
SERVICE_URL = http://seafile.itdraft.ru

[Database]
ENGINE = mysql
HOST = 127.0.0.1
PORT = 3306
USER = seafile
PASSWD = seafilepass
DB = ccnet-db
CONNECTION_CHARSET = utf8
```

В этом же файле задаются настройки для подключения LDAP-авторизации

```sh
$ sudo nano /opt/seafile/conf/ccnet.conf
...
# MS AD
[LDAP]
HOST = ldap://192.168.1.2:389/
BASE = ou=USERS,dc=itdraft,dc=ru;ou=Admins,ou=DomainUsers,ou=MSK,dc=itdraft,dc=ru
USER_DN = seafile_s@itdraft.ru
PASSWORD = mypass
#LOGIN_ATTR = uid
#LOGIN_ATTR = sAMAccountName
LOGIN_ATTR = userPrincipalName
#LOGIN_ATTR = mail
FILTER = memberOf=cn=Seafile.Access,ou=USERS,dc=itdraft,dc=ru

[LDAP_SYNC]
FIRST_NAME_ATTR = givenName
LAST_NAME_ATTR = sn
UID_ATTR = uid
```

Включим кеширование Memcached, капчу, время хранение сессии и языки

```sh
$ sudo nano /opt/seafile/conf/seahub_settings.py
# -*- coding: utf-8 -*-
SECRET_KEY = "b'o=nv+xw7xb&6jhdi^d7w^=-9j)yp@xmelvq7kh^q#8i#@^z7jd'"
SERVICE_URL = "http://seafile.itdraft.ru/"

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'seahub-db',
        'USER': 'seafile',
        'PASSWORD': 'seafilepass',
        'HOST': '127.0.0.1',
        'PORT': '3306',
        'OPTIONS': {'charset': 'utf8mb4'},
    }
}

# Enable chache
CACHES = {
    'default': {
        'BACKEND': 'django_pylibmc.memcached.PyLibMCCache',
        'LOCATION': '127.0.0.1:11211',
    },
    'locmem': {
        'BACKEND': 'django.core.cache.backends.locmem.LocMemCache',
    },
}
COMPRESS_CACHE_BACKEND = 'locmem'

# Email settings
EMAIL_USE_TLS                       = False
EMAIL_HOST                          = 'mail.example.com'
EMAIL_HOST_USER                     = 'noreply@example.com'
EMAIL_HOST_PASSWORD                 = ''
EMAIL_PORT                          = '25'
DEFAULT_FROM_EMAIL                  = EMAIL_HOST_USER
SERVER_EMAIL                        = EMAIL_HOST_USER
# https://download.seafile.com/published/seafile-manual/config/sending_email.md

TIME_ZONE                           = 'Europe/Moscow'
SITE_BASE                           = 'http://seafile.itdraft.ru'
SITE_NAME                           = 'Seafile Server'
SITE_TITLE                          = 'Seafile Server'
SITE_ROOT                           = '/'
ENABLE_SIGNUP                       = False
ACTIVATE_AFTER_REGISTRATION         = False
SEND_EMAIL_ON_ADDING_SYSTEM_MEMBER  = True
SEND_EMAIL_ON_RESETTING_USER_PASSWD = True
CLOUD_MODE                          = False
FILE_PREVIEW_MAX_SIZE               = 30 * 1024 * 1024
SESSION_COOKIE_AGE                  = 60 * 60 * 24 * 7 * 2
SESSION_SAVE_EVERY_REQUEST          = False
SESSION_EXPIRE_AT_BROWSER_CLOSE     = False

# User management options
LOGIN_REMEMBER_DAYS = 1
LOGIN_ATTEMPT_LIMIT = 3

# Other options
# Максимальное количество одновремено загружаемых файлов
MAX_NUMBER_OF_FILES_FOR_FILEUPLOAD = 20
LANGUAGE_CODE = 'ru'
LANGUAGES = (
    ('en', 'English'),
    ('ru', 'Russian'),
)

FILE_SERVER_ROOT                    = 'http://seafile.itdraft.ru/seafhttp'
```

Так же в этом файле можно прописать настройки для подключения OnlyOffice

```
...
# Enable Only Office
ENABLE_ONLYOFFICE = True
VERIFY_ONLYOFFICE_CERTIFICATE = True
ONLYOFFICE_APIJS_URL = 'http://seafile.itdraft.ru/onlyofficeds/web-apps/apps/api/documents/api.js'
ONLYOFFICE_FILE_EXTENSION = ('doc', 'docx', 'ppt', 'pptx', 'xls', 'xlsx', 'odt', 'fodt', 'odp', 'fodp', 'ods', 'fods')
ONLYOFFICE_EDIT_FILE_EXTENSION = ('docx', 'pptx', 'xlsx')
```

Дополнительные параметры можно посмотреть на [официальном сайте](https://download.seafile.com/published/seafile-manual/config/seahub_settings_py.md)

Отредактируем файл `seafile.conf`

```sh
$ sudo nano /opt/seafile/conf/seafile.conf
[fileserver]
port = 8082
max_upload_size=1024
max_download_dir_size=1024

[database]
type = mysql
host = 127.0.0.1
port = 3306
user = seafile
password = seafilepass
db_name = seafile-db
connection_charset = utf8

# Квота
[quota]
default = 5

[history]
keep_days = 7

# Очищать корзину, кол-во дней
[library_trash]
expire_days = 30

# Configure Syslog
[general]
enable_syslog = true
```

Запускаем сервисы вручную, так как в процессе первого запуска сервиса seahub надо будет указать **email** и **пароль администратора**

```sh
$ sudo su - seafile /opt/seafile/seafile-server-latest/seafile.sh start
$ sudo su - seafile /opt/seafile/seafile-server-latest/seahub.sh start
[ admin email ] admin@itdraft.ru
[ admin password ] mypasswd
[ admin password again ] mypasswd
```

Останавливаем

```sh
$ sudo su - seafile /opt/seafile/seafile-server-latest/seahub.sh stop
$ sudo su - seafile /opt/seafile/seafile-server-latest/seafile.sh stop
```

Запускаем службы и добавляем их в автозапуск

```sh
$ sudo systemctl enable --now seafile
$ sudo systemctl enable --now seahub
```

Перезапускаем Nginx

```sh
$ sudo systemctl restart nginx
```

Если надо скинуть пароль админа, выполняем команду

```sh
$ sudo su - seafile /opt/seafile/seafile-server-latest/reset-admin.sh
```
