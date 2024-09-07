---
title: "Easy Redmine - перенос на Centos 7, обновление, бэкапирование"
date: "2018-08-15"
categories: 
  - Project-Management
tags: 
  - "apache"
  - "backup"
  - "centos"
  - "easy-redmine"
  - "logrotate"
  - "mod_proxy"
  - "mysql"
  - "ruby"
  - "ssl"
image:
  path: /commons/p9lahdjjxckbx3hsm3uqaioj4bu.png
  alt: "Easy Redmine"
---

> **Easy Redmine** - это полноценное и расширяемое обновление Redmine, которое оснащено лучшими плагинами, функциями и новым дизайном для мобильных устройств. Он обеспечивает более эффективное управление проектами, четкую коммуникацию, лучший пользовательский опыт и экономит ваше драгоценное время.
{: .prompt-tip }

Есть сервер с Centos 6 и устаревшей версией Easy Redmine.  
Необходимо обновить Easy Redmine до последней версии, перенести пользовательские данные.

План работы:

- Делается бэкап данных и базы
- Разворачивается новая виртуальная машина с Centos 7
- Устанавливается необходимый софт
- Разворачивается бэкап данных на новом сервере
- Обновляется EasyRedmine

## Backup файлов и базы на старом сервере

Т.к. на старом сервере изначально было выделено мало места, устанавливаем утилиту, что примонтировать расшаренный каталог рабочего ПК.

```sh
$ sudo yum install cifs-utils
$ sudo mkdir -p /mnt/smb
$ sudo mount.cifs //192.168.1.99/share /mnt/smb -o username=user domain=DOMAIN
```

На старом сервере делаем бэкап файлов и базы Easy Redmine и сохраняем эти данные в примонтированный каталог.

```sh
$ sudo tar -zcf /mnt/smb/backup_$(date +%y%m%d).tar.gz /var/www/html/redmine
$ sudo mysqldump -u redmine -predmine redmine | gzip > /mnt/smb/backup_$(date +%y%m%d).sql.gz
```

На этом все манипуляции со старым сервером закончились.

## Разворачиваем Centos 7 и ставим софт

На новом сервере добавляем репозиторий EPEL и обновляемся

```sh
$ sudo yum install epel-release
$ sudo yum update
```

Ставим необходимый набор софта

```sh
$ sudo yum install nano htop mc wget
$ sudo yum groupinstall "Development tools"
$ sudo yum install ImageMagick ImageMagick-devel git libcurl-devel libxml2-devel libxslt-devel gcc bzip2 openssl-devel zlib-devel gdbm-devel ncurses-devel autoconf automake bison gcc-c++ libffi-devel libtool patch readline-devel sqlite-devel glibc-headers glibc-devel libyaml-devel libicu-devel libidn-devel
```

Отключаем SELinux

```sh
$ sudo setenforce 0
$ sudo nano /etc/sysconfig/selinux
SELINUX=disabled
```

Устанавливаем WEB-сервер Apache, добавляем его в автозагрузку и запускаем

```sh
$ sudo yum install httpd mod_ssl httpd-devel
$ sudo systemctl enable httpd.service
$ sudo systemctl start httpd.service
```

Открываем порты 80 и 443 в firewall

```sh
$ sudo firewall-cmd --permanent --zone=public --add-service=http
$ sudo firewall-cmd --permanent --zone=public --add-service=https
$ sudo firewall-cmd --reload
```

Устанавливаем MySQL-сервер (MariaDB) добавляем его в автозагрузку и запускаем

```sh
$ sudo yum install mariadb-server mariadb mariadb-devel
$ sudo systemctl enable mariadb.service
$ sudo systemctl start mariadb.service
```

Задаем `root-пароль` для MySQL и проверяем подключение

```sh
$ sudo mysql_secure_installation

NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

In order to log into MariaDB to secure it, we'll need the current
password for the root user.  If you've just installed MariaDB, and
you haven't set the root password yet, the password will be blank,
so you should just press enter here.

Enter current password for root (enter for none): 
OK, successfully used password, moving on...

Setting the root password ensures that nobody can log into the MariaDB
root user without the proper authorisation.

Set root password? [Y/n] 
New password: 
Re-enter new password: 
Password updated successfully!
Reloading privilege tables..
 ... Success!


By default, a MariaDB installation has an anonymous user, allowing anyone
to log into MariaDB without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.

Remove anonymous users? [Y/n] 
 ... Success!

Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n] 
 ... Success!

By default, MariaDB comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n] 
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n] 
 ... Success!

Cleaning up...

All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.

Thanks for using MariaDB!

$ sudo mysql -u root -p
```

Ставим утилиту, чтоб перенести бэкап из расшаренной папки рабочего ПК на сервер

```sh
$ sudo yum install cifs-utils
$ sudo mkdir -p /mnt/smb
$ sudo mount.cifs //192.168.1.99/share /mnt/smb -o username=user domain=DOMAIN
```

Переносим файлы на сервер

```sh
$ sudo cp /mnt/sdb/backup_180807.sql.gz /home/backup_180807.sql.gz
$ sudo cp /mnt/sdb/backup_180807.sql.gz /home/backup_180807.tar.gz
```

Добавляем пользователя `redmine` и добавляем ему права `sudo`

```sh
$ sudo adduser redmine
$ sudo passwd redmine
$ sudo visudo
## Allow root to run any commands anywhere
root    ALL=(ALL)       ALL
redmine ALL=(ALL)       ALL
```

Переключаемся на пользователя `redmine`

```sh
$ sudo su - redmine
```

Устанавливаем `Ruby 2.5.1` от пользователя `eedmine`

```sh
$ curl -sSL https://rvm.io/mpapis.asc | gpg --import -
$ curl -L get.rvm.io | bash -s stable
$ source ~/.rvm/scripts/rvm
$ rvm reload
$ rvm list known
$ rvm install 2.5
$ rvm use 2.5 --default
```

Проверяем установленную версию Ruby

```sh
$ ruby --version
```

Переносим файлы бэкапа в домашнюю папку пользователя `eedmine`

```sh
$ mv /home/backup_180807.sql.gz /home/redmine/backup_180807.sql.gz
$ mv /home/backup_180807.tar.gz /home/redmine/backup_180807.tar.gz
```

Распаковываем архив с базой и с файлами

```sh
$ gzip -d /home/redmine/backup_180807.sql.gz
$ tar -xvzf /home/redmine/backup_180807.tar.gz
```

Создаем каталог, где будет лежать EasyRedmine

```sh
$ mkdir -p /home/redmine/easyredmine/
```

Переносим распакованные файлы в него

## Настраиваем MySQL

Подключаемся к MYSQL

```sh
$ mysql -u root -p
```

Создаем базу данных, пользователя и назначаем ему пароль

```sh
> CREATE DATABASE redmine;
> CREATE USER 'redmine'@'localhost' IDENTIFIED BY 'password';
```

Назначаем привилегии на базу, обновляем привилегии, отключаемся от MySQL

```sh
> GRANT ALL PRIVILEGES ON redmine.* TO 'redmine'@'localhost';
> FLUSH PRIVILEGES;
> exit;
```

Восстанавливаем базу данных из дампа

```sh
$ mysql -u redmine -ppassword redmine < /home/redmine/backup_180807.sql
```

## Настраиваем Apache

Переключаемся обратно на пользователя `root`

```sh
$ exit
```

Добавим `vhosts` — несколько сайтов на одном ip-адресе

```sh
$ sudo nano /etc/httpd/conf.d/vhosts.conf
# Загрузка моих vhosts
IncludeOptional vhosts.d/*.conf
```

Создаем каталог, где будут лежать конфигурации `vhosts`

```sh
$ sudo mkdir /etc/httpd/vhosts.d
```

Создаем конфигурационный файл, в дальнейшем мы еще будем его редактировать

```sh
$ sudo nano /etc/httpd/vhosts.d/redmine.example.ru.conf
<VirtualHost *:80>
   ServerName redmine.example.ru

   DocumentRoot /home/redmine/easyredmine/public

   <Directory /home/redmine/easyredmine/public>
      AllowOverride all
      Options -MultiViews
      Require all granted
   </Directory>

   ErrorLog "/home/redmine/logs/redmine-error.log"
   CustomLog "/home/redmine/logs/redmine-access.log" combined
</VirtualHost>
```

Создаем каталоге, где будут лежать логи Apache

```sh
$ sudo mkdir /home/redmine/logs
```

Перезапускаем Apache

```sh
$ sudo systemctl restart httpd.service
```

Назначаем права на домашний каталог пользователя `redmine`, чтобы другие службы имели к ней доступ

```sh
$ sudo chmod o+x "/home/redmine"
```

Переключаемся на пользователя `redmine`

```sh
$ sudo su - redmine
```

Установка модуля `passenger`

```sh
$ gem install passenger
$ passenger-install-apache2-module
```

при установке модуля я оставил только Ruby

Донастраиваем Apache  
Переключаемся обратно на пользователя `root`

```sh
$ exit
```

Прописываем модуль `passenger` в Apache

```sh
$ sudo echo "LoadModule passenger_module /home/redmine/.rvm/gems/ruby-2.5.1/gems/passenger-5.3.4/buildout/apache2/mod_passenger.so" | sudo tee -a /etc/httpd/conf.modules.d/00-base.conf
```

Конфигурационный файл виртуального хоста `redmine.example.ru` приводим к виду

```sh
$ sudo cat /etc/httpd/vhosts.d/redmine.example.ru.conf
<VirtualHost *:80>
   ServerName redmine.example.ru
   DocumentRoot /home/redmine/easyredmine/public

     PassengerRoot /home/redmine/.rvm/gems/ruby-2.5.1/gems/passenger-5.3.4
     PassengerDefaultRuby /home/redmine/.rvm/gems/ruby-2.5.1/wrappers/ruby
     PassengerUser redmine

   <Directory /home/redmine/easyredmine/public>
      AllowOverride all
      Options -MultiViews
      Require all granted
   </Directory>

   ErrorLog "/home/redmine/logs/redmine-error.log"
   CustomLog "/home/redmine/logs/redmine-access.log" combined
</VirtualHost>
```

Перезапускаем Apache

```sh
$ sudo systemctl restart httpd.service
```

## Обновляем Easy Redmine

Переключаемся на пользователя `redmine`

```sh
$ sudo su - redmine
```

Скачиваем утилиту, для проверки готовности установки Easy Redmine и запускаем ее

```sh
$ wget https://raw.githubusercontent.com/easyredmine/easy_server_requirements_check/master/easycheck.sh
$ sh ./easycheck.sh
```

По результатам выполнения этой утилиты мне надо было доустановить некоторые программы: `ImageMagic`, `mysql-devel`

Устанавливаем утилиту для обновления Easy Redmine

```sh
$ gem install redmine-installer --pre
```

Запускаем процесс обновления Easy Redmine

```sh
$ redmine upgrade /home/redmine/easyredmine20180101.zip /home/redmine/easyredmine
```

Файл `easyredmine20180101.zip` был предварительно скачан с официального сайта Easy Redmine

Перезапускаем Apache

```sh
$ sudo systemctl restart httpd.service
```

## Настраиваем ротацию логов logrotate

Переключаемся обратно на пользователя `root`

```sh
$ exit
```

Логи Apache: `/home/redmine/logs ` 
Логи Easy Redmine: `/home/redmine/easyredmine/log`

Создаем конфигурационный файл

```sh
$ sudo nano /etc/logrotate.d/easyredmine
/home/redmine/logs/*log /home/redmine/easyredmine/log/*log {
  # Ежедневная ротация
  daily

  # Сжимать ротированные файлы
  compress

  # Не генерировать ошибку, если файла лога нет
  missingok

  # Если файл пустой, не выполнять никаких действий.
  notifempty

  # Ротировать 30 раз до удаления
  rotate 30

  # Пока размер лог-файла не превысит 5 мегабайт, он не будет ротироваться
  size=5M

  # Выполнять postrotate только один раз
  sharedscripts

  # Отложить сжатие последнего лога
  delaycompress

  # Создать новый лог-файл после ротирования
  create 644 root root

  # Скрипт, который необходимо выполнить после чистки лога
#  postrotate
#  /bin/systemctl reload httpd.service > /dev/null 2>/dev/null || true
#  endscript
}
```

## Бэкапирование Easy Redmine

Переключаемся на пользователя `redmine`

```sh
$ sudo su - redmine
```

Делаем бэкап средствами Redmine

```sh
$ redmine backup /home/redmine/easyredmine
```

По окончанию процесса бэкапировния будет создан каталог с файлами

```sh
$ ls /home/redmine/redmine-backups/backup_%date%_%time%
redmine.sql redmine.zip
```

Бэкапирование с помощью bash

```sh
$ sudo tar -zcf /home/redmine/redmine-backups/$(date +%y%m%d)_backup_filse.tar.gz /home/redmine/easyredmine
$ sudo mysqldump -u redmine -ppassword redmine | gzip > /home/redmine/redmine-backups/$(date +%y%m%d)_backup_base.sql.gz
```

## Настраиваем SSL

Так как Easy Redmine лежит на виртуальном сервере внутри локальной сети, то доступ наружу будет пробрасывается через модуль для Apache `mod_proxy` на сервере с внешним ip

Конфигурация для проброса Easy Redmine наружу

```sh
$ sudo cat /etc/httpd/vhosts/redmine.example.ru.conf 
<VirtualHost *:80>
 ServerName redmine.example.ru
 ServerAlias www.redmine.example.ru
 ServerAdmin root@example.ru
 Redirect / https://redmine.example.ru/
</VirtualHost>

NameVirtualHost *:443
<VirtualHost *:443>
 ServerName redmine.example.ru
 ServerAlias www.redmine.example.ru
 ServerAdmin root@example.ru
 ServerSignature Off

 SSLEngine on
 SSLCertificateFile /etc/httpd/ssl/example.ru/certificate.crt
 SSLCertificateKeyFile /etc/httpd/ssl/example.ru/private.key
 
# Enable SSL over proxy
 SSLProxyEngine On
 SSLProxyCheckPeerCN on
 SSLProxyCheckPeerExpire on
 
 ProxyRequests Off
 ProxyPreserveHost On
 ProxyVia full
 
 ProxyPass "/" "http://192.168.1.40/"
 ProxyPassReverse "/" "http://192.168.1.40/"

 # Notify the server that the connection is secure
 RequestHeader set X-Forwarded-Proto "https"
 RequestHeader set X-Forwarded-Port "443"

ErrorLog /var/www/vhosts/easyredmine/error_ssl.log
CustomLog /var/www/vhosts/easyredmine/access_ssl.log combined
</VirtualHost>
```
