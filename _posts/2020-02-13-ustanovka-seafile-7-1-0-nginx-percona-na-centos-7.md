---
title: "Установка SeaFile 7.1.0 + Nginx + Percona на Centos 7"
date: "2020-02-13"
categories: 
  - Storage-System
tags: 
  - "centos"
  - "nginx"
  - "percona"
  - "seafile"
image:
  path: /commons/978728_0af3_5.jpg
  alt: "Установка SeaFile"
---

> **SeaFile** - это кроссплатформенная система программного обеспечения для размещения файлов с открытым исходным кодом. Файлы хранятся на центральном сервере и могут быть синхронизированы с персональными компьютерами и мобильными устройствами через приложения.
{: .prompt-tip }

## Подготовка

Обновляем операционную систему, добавляем репозиторий EPEL, устанавливаем софт

```sh
$ sudo yum -y update
$ sudo yum -y install epel-release
$ sudo yum -y install nano wget net-tools
```

Устанавливаем необходимые для SeaFile пакеты

```sh
$ sudo yum -y install python3 python3-setuptools python3-pip python-ldap memcached java-1.8.0-openjdk libmemcached libreoffice-headless libreoffice-pyuno libffi-devel pwgen curl
$ sudo pip3 install --timeout=3600 Pillow pylibmc captcha jinja2 sqlalchemy psd-tools django-pylibmc django-simple-captcha
```

Переводим SELinux в режим `permissive`

```sh
$ sudo setenforce 0
$ sudo sed -i 's/^SELINUX=.*/SELINUX=permissive/' /etc/selinux/config
$ sudo getenforce
```

Запускаем службу `memcached`

```sh
$ sudo systemctl enable --now memcached
```

## Установка и настройка Nginx

Добавим репозиторий Nginx

```sh
$ sudo nano /etc/yum.repos.d/nginx.repo 
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=0
enabled=1
```

Установка web-сервера Nginx

```sh
$ sudo yum -y install nginx
```

Создадим директории для виртуальных хостов

```sh
$ sudo mkdir /etc/nginx/{sites-available,sites-enabled}
```

Отредактируем основной конфиг `nginx.conf`

```sh
$ sudo nano /etc/nginx/nginx.conf

user  nginx;
worker_processes  auto;

error_log /var/log/nginx/error.log warn;
pid /var/run/nginx.pid;
#worker_rlimit_nofile 40000;

events {
    # worker_connections 8096;
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

    server_tokens off;
    server_names_hash_bucket_size 128;
    client_max_body_size 50M;

    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    client_body_timeout 12;
    client_header_timeout 12;
    keepalive_timeout 15;
    send_timeout 10;

    # Fully disabled gzip compression to mitigate Django BREACH attack: https://www.djangoproject.com/weblog/2013/aug/06/breach-and-django/
    gzip off;
    #gzip_vary on;
    #gzip_proxied expired no-cache no-store private auth any;
    #gzip_comp_level 9;
    #gzip_min_length 10240;
    #gzip_buffers 16 8k;
    #gzip_http_version 1.1;
    #gzip_types text/plain text/css text/xml text/javascript application/javascript application/x-javascript application/xml font/woff2;
    #gzip_disable "MSIE [1-6].";

    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*.conf;
}
```

Отредактируем конфиг `seafile.conf`

```sh
$ sudo nano /etc/nginx/sites-available/seafile.conf
log_format seafileformat '$http_x_forwarded_for $remote_addr [$time_local] "$request" $status $body_bytes_sent "$http_referer" "$http_user_agent" $upstream_response_time';

server {
    listen 80;
    server_name seafile.example.com;

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

Создаем сим линк, чтобы активировать конфиг `seafile.conf`

```sh
$ sudo ln -s /etc/nginx/sites-available/seafile.conf /etc/nginx/sites-enabled/seafile.conf
```

Проверяем конфиги Nginx на ошибки

```sh
$ sudo nginx -t
```

Добавляем Nginx в автозагрузку и запускаем web-сервер

```sh
$ sudo systemctl enable --now nginx
```

## Firewall

Настройка firewall, открываем порты

```sh
$ sudo firewall-cmd --zone=public --add-service={http,https} --permanent
$ sudo firewall-cmd --reload
```

## Установка сервера базы данных Percona

Добавим репозиторий Percona

```sh
$ sudo yum -y install https://repo.percona.com/yum/percona-release-latest.noarch.rpm
```

Устанавливаем Percona-Server

```sh
$ sudo yum -y install Percona-Server-server-57
```

Запускаем сервис и добавляем его в автозагрузку

```sh
$ sudo systemctl enable --now mysqld
```

Ищем пароль, который сгенерировался при установке и меняем его

```sh
$ sudo grep -i password /var/log/mysqld.log
$ sudo mysql_secure_installation
New password:
Re-enter new password:
Change the password for root ? ((Press y|Y for Yes, any other key for No) :
Remove anonymous users? (Press y|Y for Yes, any other key for No) : y
Disallow root login remotely? (Press y|Y for Yes, any other key for No) : y
Remove test database and access to it? (Press y|Y for Yes, any other key for No) : y
Reload privilege tables now? (Press y|Y for Yes, any other key for No) : y
```

Что б в дальнейшем не было проблем с паролем для пользователя `seafile`, изменим политику паролей в PerconaDB

```sh
$ mysql -u root -p
> SHOW GLOBAL VARIABLES LIKE 'validate_password%';
> SET GLOBAL validate_password_special_char_count = 0;
> flush privileges;
> quit;
```

## Установка Seafile

Создаем пользователя `seafile`

```sh
$ sudo useradd -m -U -r -d /opt/seafile seafile
$ sudo chmod 750 /opt/seafile
```

Добавляем пользователя `nginx` в группу `seafile`

```sh
$ sudo usermod -aG seafile nginx
```

Переключаемся на пользователя `seafile`

```sh
$ sudo su - seafile
```

Создаем директорию, куда потом перенесем архив с дистрибутивом

```sh
$ mkdir -p /opt/seafile/installed
```

Переходим в директорию, где будет установлен SeaFile

```sh
$ cd
```

Скачиваем архив, распаковываем его

```sh
$ curl -OL https://download.seadrive.org/seafile-server_7.1.0_x86-64.tar.gz
$ tar xzf seafile-server_7.1.0_x86-64.tar.gz
```

Перемещаем архив в каталог `installed`

```sh
$ mv seafile-server_7.1.0_x86-64.tar.gz installed
```

Запускаем установку SeaFile

```sh
$ ./seafile-server-7.1.0/setup-seafile-mysql.sh

Press ENTER to continue
[ server name ] seafile
[ This server's ip or domain ] seafile.example.com
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
[ root password ]

verifying password of user root ...  done

Enter the name for mysql user of seafile. It would be created if not exists.
[ default "seafile" ] 

Enter the password for mysql user "seafile":
(тут лучше использовать пароль без спец.символов @#$%^)
[ password for seafile ]

Enter the database name for ccnet-server:
[ default "ccnet-db" ] 

Enter the database name for seafile-server:
[ default "seafile-db" ] 

Enter the database name for seahub:
[ default "seahub-db" ] 

This is your configuration
    server name:            seafile
    server ip/domain:       seafile.example.com

    seafile data dir:       /opt/seafile/seafile-data
    fileserver port:        8082

    database:               create new
    ccnet database:         ccnet-db
    seafile database:       seafile-db
    seahub database:        seahub-db
    database user:          seafile
```

Если что-то пошло не так, команды для управления базами/пользователями `perconadb`

```sh
$ mysql -u root -p
> SHOW databases;
> SELECT User,mysql_native_passwor,Host FROM mysql.user;
> SELECT User,Host FROM mysql.user;
> SHOW GRANTS FOR 'seafile'@'localhost';
> REVOKE ALL PRIVILEGES, GRANT OPTION FROM 'seafile'@'localhost';
> DROP USER 'seafile'@'localhost';
> DROP DATABASE `ccnet-db`;
> DROP DATABASE `seafile-db`;
> DROP DATABASE `seahub-db`;
```

Переключаемся на предыдущего пользователя (правами `sudo`)

```sh
$ exit
```

Создаем файл `seafile.service`

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

Создаем файл `seahub.service`

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

Скрипт для перезапуска SeaFile

```sh
$ sudo nano /usr/local/sbin/seafile-server-restart
#!/bin/bash
for ACTION in stop start ; do
    for SERVICE in seafile seahub ; do
      systemctl ${ACTION} ${SERVICE}
    done
done
```

Назначим права

```sh
$ sudo chmod 700 /usr/local/sbin/seafile-server-restart
```

## Настройка SeaFile

Отредактируем `seafdav.conf`, для активации `webdav`

```sh
$ sudo nano /opt/seafile/conf/seafdav.conf
[WEBDAV]
enabled = true
port = 8080
fastcgi = true
share_name = /seafdav
```

Проверяем настройки `ccnet.conf`

```sh
$ sudo nano /opt/seafile/conf/ccnet.conf
[General]
SERVICE_URL = http://seafile.example.com
...
```

Включим кеширование Memcached, капчу, время хранение сессии и языки (добавляем после строчек, которые были в файле).

```sh
$ sudo nano /opt/seafile/conf/seahub_settings.py
...
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
SITE_BASE                           = 'http://seafile.example.com'
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
MAX_NUMBER_OF_FILES_FOR_FILEUPLOAD = 10
LANGUAGE_CODE = 'ru'
LANGUAGES = (
    ('en', 'English'),
    ('ru', 'Russian'),
)

FILE_SERVER_ROOT                    = 'http://seafile.example.com/seafhttp'
```

Дополнительные параметры можно посмотреть на [официальном сайте](https://download.seafile.com/published/seafile-manual/config/seahub_settings_py.md)

Отредактируем файл `seafile.conf`

```sh
$ sudo nano /opt/seafile/conf/seafile.conf
[fileserver]
port = 8082
max_upload_size=64
max_download_dir_size=64

[quota]
default = 2

[history]
keep_days = 7

# Очищать корзину, кол-во дней
[library_trash]
expire_days = 30

# Configure Syslog
[general]
enable_syslog = true
...
```

В процессе первого запуска задаем имя и пароль администратора.

```sh
$ sudo su - seafile /opt/seafile/seafile-server-latest/seafile.sh start
$ sudo su - seafile /opt/seafile/seafile-server-latest/seahub.sh start
[ admin email ] admin@example.com
[ admin password ] 
[ admin password again ]
```

Останавливаем

```sh
$ sudo su - seafile /opt/seafile/seafile-server-latest/seahub.sh stop
$ sudo su - seafile /opt/seafile/seafile-server-latest/seafile.sh stop
```

Добавляем службы в автозапуск и запускаем их

```sh
$ sudo systemctl daemon-reload
$ sudo systemctl enable --now seafile
$ sudo systemctl enable --now seahub
```

Если надо скинуть пароль админа, выполняем команду

```sh
$ sudo su - seafile /opt/seafile/seafile-server-latest/reset-admin.sh
```

## UPD 2021.06.01

При очередной установке, при выполнении команды `sudo pip3 install ...` появлялась ошибка связанная с отсутствия нужных утилит

Решение:
```sh
$ sudo yum -y install gcc gcc-c++ python36-devel
```
