---
title: "Установка ITSM-системы GLPI + NGINX + PHP 7.4 + Percona в Centos 7"
date: "2020-02-14"
categories: 
  - Asset-Management
  - Database-System
tags: 
  - "centos"
  - "glpi"
  - "mysql"
  - "nginx"
  - "percona"
  - "php"
  - "php-fpm"
image:
  path: /commons/424864_ab6e.jpg
  alt: "Установка ITSM-системы GLPI"
---

> **GLPI** это ITSM-система, которая позволит вам легко управлять и планировать IT изменения, быстро решать проблемы с помощью автоматизации.  
> GLPI является системой работы с заявками и инцидентами, а также для инвентаризации компьютерного оборудования.
{: .prompt-tip }

## Подготовка

Обновляем операционную систему, добавляем репозиторий EPEL, устанавливаем софт

```sh
$ sudo yum -y update
$ sudo yum -y install epel-release
$ sudo yum -y install nano wget net-tools
```

## Selinux

Переводим SELinux в режим `permissive`

```sh
$ sudo setenforce 0
$ sudo getenforce
$ sudo sed -i 's/^SELINUX=.*/SELINUX=permissive/' /etc/selinux/config
```

Применяем так же:

```sh
$ sudo setsebool -P httpd_can_network_connect on
$ sudo setsebool -P httpd_can_network_connect_db on
$ sudo setsebool -P httpd_can_sendmail on
```

## Установка и настройка PHP 7.4

Установим `yum-utils` для инструмента `yum-config-manager`

```sh
$ sudo yum -y install yum-utils
```

Включаем REMI репозиторий для установки PHP 7.4

```sh
$ sudo rpm -Uvh http://rpms.famillecollet.com/enterprise/remi-release-7.rpm
$ sudo yum-config-manager --enable remi-php74
```

Устанавливаем PHP 7.4 и некоторые компоненты

```sh
$ sudo yum -y install php php-xml php-soap php-xmlrpc php-mbstring php-json php-gd php-pecl-mcrypt php-common php-fpm php-pdo php-mysqlnd php-imap php-embedded php-ldap php-odbc php-zip php-fileinfo php-process php-opcache php-curl php-intl php-pear php-imagick php-memcache php-pspell php-gettext php-apcu php-pecl-recode php-tidy php-xsl php-pear-CAS
```

Меняем настройки в конфигурационных файлах

```sh
$ sudo sed -i 's/^short_open_tag = .*/short_open_tag = On/' /etc/php.ini
$ sudo sed -i 's/^date.timezone = .*/date.timezone = Europe/Moscow/' /etc/php.ini
```

```sh
$ sudo sed -i 's/^opcache.revalidate_freq= .*/opcache.revalidate_freq=0/' /etc/php.d/10-opcache.ini
```

```sh
$ sudo sed -i 's/^user = .*/user = nginx/' /etc/php-fpm.d/www.conf
$ sudo sed -i 's/^group = .*/group = nginx/' /etc/php-fpm.d/www.conf
$ sudo sed -i 's/^listen = .*/listen = /var/run/php-fpm/php-fpm.sock/' /etc/php-fpm.d/www.conf
$ sudo sed -i 's/^listen.owner = .*/listen.owner = nginx/' /etc/php-fpm.d/www.conf
$ sudo sed -i 's/^listen.group = .*/listen.group = nginx/' /etc/php-fpm.d/www.conf
```

Добавляем PHP-FPM в автозагрузку

```sh
$ sudo systemctl enable php-fpm
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

Установка Web-сервера Nginx

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

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;

events {
    worker_connections  1024;
    use epoll;
    multi_accept on;
}


http {
    server_tokens off;
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    sendfile_max_chunk 128k;
    tcp_nopush     on;
    tcp_nodelay on;

    keepalive_timeout  65;
    reset_timedout_connection on;
    client_header_timeout 3;
    client_body_timeout 5;
    send_timeout 3;
    client_header_buffer_size 2k;
    client_body_buffer_size 256k;
    client_max_body_size 12m;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*.conf;
}
```

Отредактируем конфиг `glpi.conf`

```sh
$ sudo nano /etc/nginx/sites-available/glpi.conf
server {
    listen      80;
    server_name glpi.example.com;
    index       index.php index.html;

    access_log  /var/log/nginx/glpi.gge.lan.access.log combined;
    error_log   /var/log/nginx/glpi.gge.lan.error.log error;

    set $php_sock unix:/var/run/php-fpm/php-fpm.sock;
    set $root_path /opt/glpi;
    root $root_path;

    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Host $host;

    client_max_body_size 1024M;
    client_body_buffer_size 4M;

    location = /favicon.ico {
        log_not_found off;
        access_log off;
    }

    location = /robots.txt {
        allow all;
        log_not_found off;
        access_log off;
    }

    location / {
        try_files $uri $uri/ /index.php?$args;

        if (!-e $request_filename)
        {
            rewrite ^(.+)$ /index.php?q=$1 last;
        }
 
        location ~* ^.+\.(jpeg|jpg|png|gif|bmp|ico|svg|css|js)$ {
            expires     max;
        }
 
        location ~ [^/]\.php(/|$) {
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
            if (!-f $document_root$fastcgi_script_name) {
                return  404;
            }

            fastcgi_pass    $php_sock;
            fastcgi_index   index.php;
            include         fastcgi_params;
            fastcgi_intercept_errors on;
            fastcgi_ignore_client_abort off;
            fastcgi_connect_timeout 60;
            fastcgi_send_timeout 180;
            fastcgi_read_timeout 180;
            fastcgi_buffer_size 128k;
            fastcgi_buffers 4 256k;
            fastcgi_busy_buffers_size 256k;
            fastcgi_temp_file_write_size 256k;
            fastcgi_param PHP_VALUE "
                memory_limit = 256M
                file_uploads = on
                max_execution_time = 600
                session.auto_start = off
                session.use_trans_sid = 0
            ";
        }
    }

    location ~* "/\.(htaccess|htpasswd)$" {
        deny    all;
        return  404;
    }
}
```

Создаем сим линк, чтобы активировать конфиг `glpi.conf`

```sh
$ sudo ln -s /etc/nginx/sites-available/glpi.conf /etc/nginx/sites-enabled/glpi.conf
```

Проверяем конфиг

```sh
$ sudo nginx -t
```

Добавляем nginx в автозагрузку

```sh
$ sudo systemctl enable nginx
```

## Firewall

Настройка Firewall, открываем порты

```sh
$ sudo firewall-cmd --zone=public --add-service={http,https} --permanent
$ sudo firewall-cmd --reload
```

## Percona

Добавим репозиторий Percona

```sh
$ sudo yum -y install https://repo.percona.com/yum/percona-release-latest.noarch.rpm
```

Устанавливаем Percona Server

```sh
$ sudo yum -y install Percona-Server-server-57
```

Запускаем сервис и добавляем его в автозагрузку

```sh
$ sudo systemctl enable --now mysqld
```

Ищем пароль, который сгенерировался автоматом и меняем его

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

## Установка GLPI

Создаем пользователя и базу

```sh
$ sudo mysql -u root -p
> CREATE DATABASE glpidb;
> GRANT ALL PRIVILEGES ON glpidb.* TO 'glpiuser'@'localhost' IDENTIFIED BY 'password';
> FLUSH PRIVILEGES;
> exit;
```

Скачиваем и распаковываем GLPI

```sh
$ cd
$ wget -c https://github.com/glpi-project/glpi/releases/download/9.4.5/glpi-9.4.5.tgz
$ tar -xf glpi-9.4.5.tgz
$ sudo mv glpi /opt/
$ sudo chmod 755 -R /opt/glpi/
$ sudo chown nginx:nginx -R /opt/glpi/
```

Запускаем PHP-FPM и запускаем Web-сервер

```sh
$ sudo systemctl restart php-fpm
$ sudo systemctl restart nginx
```

Открываем браузер и переходим по адресу `http://glpi.example.com`

По умолчанию логины / пароли:

> - `glpi` / `glpi` для учетной записи администратора
> - `tech` / `tech` для технической учетной записи
> - `normal` / `normal` для обычной учетной записи
> - `post-only` / `postonly` только для подачи заявок
{: .prompt-info }
