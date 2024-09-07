---
title: "Установка Redmine 2.4.0"
date: "2016-03-01"
categories: 
  - Project-Management 
tags: 
  - "centos"
  - "redmine"
image:
  path: /commons/mining_decline_2-min.png
  alt: "Установка Redmine"
---

> **Redmine** - это открытое серверное веб-приложение для управления проектами и задачами (в том числе для отслеживания ошибок). Разработано на языке Ruby и использует веб-фреймворк Ruby on Rails. Это гибкое веб-приложение, которое включает в себя диаграммы Ганта, календарь, вики, форумы, настройку ролей и уведомления по электронной почте.
{: .prompt-tip }

Устанавливаем необходимые библиотеки

```sh
$ sudo yum install make gcc gcc-c++ zlib-devel curl-devel openssl-devel httpd-devel apr-devel apr-util-devel mysql-devel
$ sudo yum install zlib zlib-devel openssl-devel sqlite-devel gcc-c++ glibc-headers libyaml-devel readline readline-devel zlib-devel libffi-devel
```

Скачиваем исходники Ruby

```sh
$ wget http://cache.ruby-lang.org/pub/ruby/2.1/ruby-2.1.5.tar.gz
```

Распаковываем

```sh
$ tar zxvf ruby-2.1.5.tar.gz
```

Компилируем и устанавливаем

```sh
$ cd ruby-2.1.5
$ ./configure
$ make
$ make install
```

Смотрим версию

```sh
$ ruby -v
```

Устанавливаем passenger

```sh
$ sudo gem install passenger
$ sudo passenger-install-apache2-module
```

Создаем конфигурационный файл

```sh
$ sudo nano /etc/httpd/conf.d/passenger.conf

LoadModule passenger_module /usr/local/lib/ruby/gems/2.1.0/gems/passenger-5.0.4/buildout/apache2/mod_passenger.so
<IfModule mod_passenger.c>
   PassengerRoot /usr/local/lib/ruby/gems/2.1.0/gems/passenger-5.0.4
   PassengerDefaultRuby /usr/local/bin/ruby
</IfModule>
```

Перезапускаем Apache

```sh
$ sudo service httpd restart
```

Настройки хоста для Apache:

```
<VirtualHost *:80>
ServerName www.yourhost.com
# !!! Be sure to point DocumentRoot to 'public'!
DocumentRoot /somewhere/public 
<Directory /somewhere/public>
# This relaxes Apache security settings.
AllowOverride all
# MultiViews must be turned off.
Options -MultiViews
# Uncomment this if you're on Apache >= 2.4:
#Require all granted
</Directory>
</VirtualHost>
```

Качаем Redmine

```sh
$ cd ~
$ wget http://www.redmine.org/releases/redmine-2.4.0.tar.gz
```

Распаковываем

```sh
$ tar zxvf redmine-2.4.0.tar.gz
```

Переносим распакованные файлы в `/var/www/html/redmine`

```sh
$ sudo mkdir /var/www/redmine
$ sudo mv redmine-2.4.0/* /var/www/redmine
```

Ставим

```sh
$ sudo gem install bundle
```

Меняем владельца директории

```sh
$ sudo chown -R apache:apache /var/www/html/redmine
```

Доустанавливаем библиотеки

```sh
$ sudo yum install ImageMagick-devel
$ sudo gem install rmagick -v '2.13.2'
```

Устанавливаем redmine

```sh
$ cd /var/www/redmine
$ sudo bundle install --without postgresql sqlite test development
```

Настройка подключения к базе

```sh
$ mysql -u root -p
> create database redmine character set utf8;
> grant all privileges on redmine.* to 'redmine'@'localhost' identified by 'redmine';
> flush privileges;
> quit;
```

Конфигурируем Redmine для подключения к базе

```sh
$ cd /var/www/html/redmine/config
$ sudo cp database.yml.example database.yml
```

Открываем `database.yml` и прописываем логин/пароль от базы

```
$ sudo nano database.yml
```

переходим в каталог и доустанавливаем

```sh
$ sudo cd /var/www/html/redmine
$ sudo bundle install
$ sudo rake generate_secret_token
```

Первичное заполнение базы

```sh
$ sudo rake db:migrate RAILS_ENV="production"
$ sudo rake redmine:load_default_data RAILS_ENV="production"
```

Установка плагинов

```sh
$ cd /var/www/html/redmine/plugins
```

- плагин `redmine_multiprojects_issue`

```
$ sudo wget https://github.com/nanego/redmine_multiprojects_issue/archive/master.zip
$ sudo unzip masters.zip
$ sudo bundle install
$ sudo rake redmine:plugins:migrate RAILS_ENV=production
$ sudo rm master.zip
```

- плагин `redmine_base_select2`

```
$ sudo wget https://github.com/jbbarth/redmine_base_select2/archive/master.zip
$ sudo unzip masters.zip
$ sudo bundle install
$ sudo rake redmine:plugins:migrate RAILS_ENV=production
$ sudo rm master.zip
```

- плагин `redmine_base_deface`

```
$ sudo wget https://github.com/jbbarth/redmine_base_deface/archive/master.zip
$ sudo unzip masters.zip
$ sudo bundle install
$ sudo rake redmine:plugins:migrate RAILS_ENV=production
$ sudo rm master.zip
```

Перезапускаем Apache

```sh
$ sudo service httpd restart
```
