---
title: "Установка Redmine 4.0.5 NGINX PostgreSQL в Centos 7"
date: "2019-12-10"
categories: 
  - Project-Management
tags: 
  - "centos"
  - "nginx"
  - "postgresql"
  - "redmine"
  - "ldap"
  - "freeipa"
image:
  path: /commons/1039062_e553_4.jpg
  alt: "Установка Redmine"
---

> **Redmine** — открытое серверное веб-приложение для управления проектами и задачами (в том числе для отслеживания ошибок). Redmine написан на Ruby и представляет собой приложение на основе широко известного веб-фреймворка Ruby on Rails
{: .prompt-tip }

## Подготовительный этап

Устанавливаем пакеты, необходимые для сборки Redmine и Ruby из исходного кода

```sh
$ sudo yum install nano curl gpg gcc gcc-c++ make patch autoconf automake bison libffi-devel libtool libcurl-devel
$ sudo yum install readline-devel sqlite-devel zlib-devel openssl-devel readline  glibc-headers glibc-devel
$ sudo yum install zlib libyaml-devel bzip2 iconv-devel ImageMagick ImageMagick-devel gdbm-devel  mariadb-devel
```

Создадим нового пользователя, добавляем его в группу `wheel`

```sh
$ sudo useradd -m -U -r -d /opt/redmine redmine
$ sudo usermod -aG wheel redmine
$ sudo chmod 750 /opt/redmine
```

Разрешаем пользователю redmine делать sudo, не запрашивая пароль

```sh
$ echo "redmine ALL=(ALL) NOPASSWD: ALL" | sudo tee -a /etc/sudoers.d/redmine > /dev/null
```

Добавляем правила в Firewall

```sh
$ sudo firewall-cmd --permanent --zone=public --add-service=http
$ sudo firewall-cmd --permanent --zone=public --add-service=https 
$ sudo firewall-cmd --reload
$ sudo firewall-cmd --list-all-zones
```

## Установка и настройка PostgreSQL 10

Добавляем репозиторий

```sh
$ sudo yum install https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm
```

Устанавливаем PostgreSQL

```sh
$ sudo yum install libpqxx libpqxx-devel postgresql10.x86_64 postgresql10-server postgresql10-contrib postgresql10-libs postgresql10-tcl postgresql10-devel protobuf-devel
```

Инициализируем пространство для БД

```sh
$ sudo /usr/pgsql-10/bin/postgresql-10-setup initdb
```

Добавляем службу в автозагрузку и запускаем PostgreSQL

```sh
$ sudo systemctl start postgresql-10
$ sudo systemctl enable postgresql-10
```

Переключаемся на пользователя postgres

```sh
$ sudo su - postgres
```

Создаем пользователя БД

```sh
-bash-4.2$ createuser redmine
```

Переключаемся на консольный клиент `psql`

```sh
-bash-4.2$ psql
```

Задаем пароль для пользователя БД

```sh
postgres=# ALTER USER redmine WITH ENCRYPTED password '%passwd%';
```

Создам базу и задаем владельца базы

```sh
postgres=# CREATE DATABASE redmine WITH ENCODING='UTF8' OWNER=redmine;
Exit
\q
logout
```

Настраиваем доступ к PostgreSQL

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

Перезапускаем PostgreSQL

```sh
$ sudo systemctl restart postgresql-10
```

## Установка Ruby из исходников

Переключаемся на пользователя `redmine`

```sh
$ sudo su - redmine
```

Скачиваем `ruby 2.6.5`, собираем из исходников

```sh
$ curl -L https://cache.ruby-lang.org/pub/ruby/2.6/ruby-2.6.5.tar.gz -o /tmp/ruby-2.6.5.tar.gz 
$ cd /tmp
$ tar zxf /tmp/ruby-2.6.5.tar.gz
$ sudo mv /tmp/ruby-2.6.5 /opt/ruby
$ sudo chown -R redmine. /opt/ruby
$ cd /opt/ruby
$ ./configure --disable-install-doc
$ make -j2
$ sudo make install
```

Что бы удалить собранный из исходников ruby

```sh
$ sudo make clean
```

Проверяем

```sh
$ ruby -v
ruby 2.6.5p114 (2019-10-01 revision 67812) [x86_64-linux]
```

## Установка Rubygems из исходников

Скачиваем `rubygems 3.0.6`, собираем из исходников

```sh
$ curl -L https://rubygems.org/rubygems/rubygems-3.0.6.tgz -o /tmp/rubygems-3.0.6.tgz
$ cd /tmp
$ tar -zxf rubygems-3.0.6.tgz
$ cd rubygems-3.0.6
$ sudo chown -R redmine. /usr/local/lib/ruby/
$ alias sudo='sudo env PATH=$PATH'
$ sudo ruby setup.rb
```

Проверяем

```sh
$ gem -v
3.0.6
```

## Устанавливаем Redmine. Начало

Скачиваем и распаковываем `redmine`

```sh
$ cd /tmp
$ curl -L http://www.redmine.org/releases/redmine-4.0.5.tar.gz -o /tmp/redmine-4.0.5.tar.gz
$ tar zxf /tmp/redmine-4.0.5.tar.gz
$ sudo mv /tmp/redmine-4.0.5 /opt/redmine/redmine-4.0.5
```

Создаем сим линк

```sh
$ sudo ln -s /opt/redmine/redmine-4.0.5 /opt/redmine/redmine-latest
$ sudo chown -h redmine:redmine /opt/redmine/redmine-latest
```

Настраиваем подключение к PostgreSQL

```sh
$ cp /opt/redmine/redmine-4.0.5/config/database.yml.example /opt/redmine/redmine-4.0.5/config/database.yml
$ nano /opt/redmine/redmine-4.0.5/config/database.yml

production:
  adapter: postgresql
  database: redmine
  host: localhost
  username: redmine
  password: "%passwd%"
  encoding: utf8
```

Устанавливаем Redmine (если сервер имеет выход в интернет)

```sh
$ alias sudo='sudo env PATH=$PATH'
$ cd /opt/redmine/redmine-4.0.5
$ sudo gem install bundler
```

Если сервер не имеет выход в интернет, надо скачать необходимые пакеты (gem)

```
gem "bundler", ">= 1.5.0" https://rubygems.org/downloads/bundler-2.0.2.gem
gem "rails", "5.2.3" https://rubygems.org/downloads/rails-5.2.3.gem
gem "rouge", "~> 3.3.0" https://rubygems.org/downloads/rouge-3.3.0.gem
gem "request_store", "1.0.5" https://rubygems.org/downloads/request_store-1.0.5.gem
gem "mini_mime", "~> 1.0.1" https://rubygems.org/downloads/mini_mime-1.0.1.gem
gem "actionpack-xml_parser" https://rubygems.org/downloads/actionpack-xml_parser-2.0.1.gem
gem "roadie-rails", "~> 1.3.0" https://rubygems.org/downloads/roadie-rails-1.3.0.gem
gem "mimemagic" https://rubygems.org/downloads/mimemagic-0.3.3.gem
gem "mail", "~> 2.7.1" https://rubygems.org/downloads/mail-2.7.1.gem
gem "csv", "~> 3.0.1" if RUBY_VERSION >= "2.3" && RUBY_VERSION < "2.6" https://rubygems.org/downloads/csv-3.1.0.gem
gem "nokogiri", (RUBY_VERSION >= "2.3" ? "~> 1.10.0" : "~> 1.9.1") https://rubygems.org/downloads/nokogiri-1.10.5.gem
gem "i18n", "~> 0.7.0" https://rubygems.org/downloads/i18n-0.7.0.gem
gem "xpath", "< 3.2.0" if RUBY_VERSION < "2.3" https://rubygems.org/downloads/xpath-3.2.0.gem
gem "sprockets", "~> 3.7.2" https://rubygems.org/downloads/sprockets-3.7.2.gem
gem 'tzinfo-data', platforms: [:mingw, :x64_mingw, :mswin] https://rubygems.org/downloads/tzinfo-data-1.2019.3.gem
gem "rbpdf", "~> 1.19.6" https://rubygems.org/downloads/rbpdf-1.20.1.gem
gem "net-ldap", "~> 0.16.0" https://rubygems.org/downloads/net-ldap-0.16.1.gem
gem "ruby-openid", "~> 2.9.2", :require => "openid" https://rubygems.org/downloads/ruby-openid-2.9.2.gem
gem "rack-openid" https://rubygems.org/downloads/rack-openid-1.4.2.gem
gem "rmagick", "~> 2.16.0" https://rubygems.org/downloads/rmagick-2.16.0.gem
gem "redcarpet", "~> 3.4.0" https://rubygems.org/downloads/redcarpet-3.4.0.gem
gem "pg", "~> 1.1.4", :platforms => [:mri, :mingw, :x64_mingw] https://rubygems.org/downloads/pg-1.1.4.gem
gem 'mini_portile2' (~> 2.4.0)  https://rubygems.org/downloads/mini_portile2-2.4.0.gem
gem 'tzinfo' (>= 1.0.0) https://rubygems.org/downloads/tzinfo-2.0.0.gem
gem 'concurrent-ruby' (~> 1.0) https://rubygems.org/downloads/concurrent-ruby-1.0.5.gem
gem 'sprockets-rails' (>= 2.0.0) https://rubygems.org/downloads/sprockets-rails-3.2.1.gem
#gem 'railties' (= 6.0.1) https://rubygems.org/downloads/railties-6.0.1.gem
#gem 'actiontext' (= 6.0.1) https://rubygems.org/downloads/actiontext-6.0.1.gem
#gem 'actionmailbox' (= 6.0.1) https://rubygems.org/downloads/actionmailbox-6.0.1.gem
#gem 'activestorage' (= 6.0.1) https://rubygems.org/downloads/activestorage-6.0.1.gem
#gem 'actioncable' (= 6.0.1) https://rubygems.org/downloads/actioncable-6.0.1.gem
#gem 'activejob' (= 6.0.1) https://rubygems.org/downloads/activejob-6.0.1.gem
#gem 'actionmailer' (= 6.0.1) https://rubygems.org/downloads/actionmailer-6.0.1.gem
#gem 'activerecord' (= 6.0.1) https://rubygems.org/downloads/activerecord-6.0.1.gem
#gem 'activemodel' (= 6.0.1) https://rubygems.org/downloads/activemodel-6.0.1.gem
#gem 'actionview' (= 6.0.1) https://rubygems.org/downloads/actionview-6.0.1.gem
#gem 'actionpack' (= 6.0.1) https://rubygems.org/downloads/actionpack-6.0.1.gem
#gem 'activesupport' (= 6.0.1) https://rubygems.org/downloads/activesupport-6.0.1.gem
gem 'method_source' (>= 0) https://rubygems.org/downloads/method_source-0.9.2.gem
gem 'thor' (>= 0.20.3, < 2.0) https://rubygems.org/downloads/thor-0.20.3.gem
gem 'marcel' (~> 0.3.1) https://rubygems.org/downloads/marcel-0.3.1.gem
gem 'websocket-driver' (>= 0.6.1) https://rubygems.org/downloads/websocket-driver-0.7.1.gem
gem 'nio4r' (~> 2.0) https://rubygems.org/downloads/nio4r-2.0.0.gem
gem 'globalid' (>= 0.3.6) https://rubygems.org/downloads/globalid-0.4.2.gem
gem 'rails-dom-testing' (~> 2.0) https://rubygems.org/downloads/rails-dom-testing-2.0.3.gem
gem 'rails-html-sanitizer' (~> 1.1, >= 1.2.0) https://rubygems.org/downloads/rails-html-sanitizer-1.3.0.gem
gem 'erubi' (~> 1.4) https://rubygems.org/downloads/erubi-1.4.0.gem
gem 'builder' (~> 3.1) https://rubygems.org/downloads/builder-3.1.4.gem
gem 'rack-test' (>= 0.6.3) https://rubygems.org/downloads/rack-test-1.1.0.gem
gem 'zeitwerk' (~> 2.2) https://rubygems.org/downloads/zeitwerk-2.2.1.gem
gem 'tzinfo' (~> 1.1) https://rubygems.org/downloads/tzinfo-1.1.0.gem
gem 'websocket-extensions' (>= 0.1.0) https://rubygems.org/downloads/websocket-extensions-0.1.4.gem
gem 'loofah' (~> 2.3) https://rubygems.org/downloads/loofah-2.3.1.gem
gem 'crass' (~> 1.0.2) https://rubygems.org/downloads/crass-1.0.5.gem
gem 'thread_safe' (~> 0.1) https://rubygems.org/downloads/thread_safe-0.1.3.gem
gem 'atomic' (>= 0) https://rubygems.org/downloads/atomic-1.1.101.gem
gem 'railties' (= 5.2.3) https://rubygems.org/downloads/railties-5.2.3.gem
gem 'actiontext' (= 5.2.3) https://rubygems.org/downloads/actiontext-5.2.3.gem
gem 'actionmailbox' (= 5.2.3) https://rubygems.org/downloads/actionmailbox-5.2.3.gem
gem 'activestorage' (= 5.2.3) https://rubygems.org/downloads/activestorage-5.2.3.gem
gem 'actioncable' (= 5.2.3) https://rubygems.org/downloads/actioncable-5.2.3.gem
gem 'activejob' (= 5.2.3) https://rubygems.org/downloads/activejob-5.2.3.gem
gem 'actionmailer' (= 5.2.3) https://rubygems.org/downloads/actionmailer-5.2.3.gem
gem 'activerecord' (= 5.2.3) https://rubygems.org/downloads/activerecord-5.2.3.gem
gem 'activemodel' (= 5.2.3) https://rubygems.org/downloads/activemodel-5.2.3.gem
gem 'actionview' (= 5.2.3) https://rubygems.org/downloads/actionview-5.2.3.gem
gem 'actionpack' (= 5.2.3) https://rubygems.org/downloads/actionpack-5.2.3.gem
gem 'activesupport' (= 5.2.3) https://rubygems.org/downloads/activesupport-5.2.3.gem
gem 'arel' (>= 9.0) https://rubygems.org/downloads/arel-9.0.0.gem
gem 'minitest' (~> 5.1) https://rubygems.org/downloads/minitest-5.1.0.gem
gem 'roadie' (~> 3.1) https://rubygems.org/downloads/roadie-3.1.1.gem
gem 'css_parser' (~> 1.3.4) https://rubygems.org/downloads/css_parser-1.3.4.gem
gem 'nokogiri' (>= 1.5.0, < 1.7.0) https://rubygems.org/downloads/nokogiri-1.6.8.1.gem
gem 'addressable' (>= 0) https://rubygems.org/downloads/addressable-2.7.0.gem
gem 'public_suffix' (>= 2.0.2, < 5.0) https://rubygems.org/downloads/public_suffix-4.0.1.gem
gem 'mini_portile2' (~> 2.1.0) https://rubygems.org/downloads/mini_portile2-2.1.0.gem
gem 'sprockets' (~> 3.7.2 ) https://rubygems.org/downloads/sprockets-3.7.2.gem
gem 'rbpdf (~> 1.19.6)' https://rubygems.org/downloads/rbpdf-1.19.8.gem
gem 'rbpdf-font' (~> 1.19.0) https://rubygems.org/downloads/rbpdf-font-1.19.1.gem
gem 'htmlentities' (>= 0) https://rubygems.org/downloads/htmlentities-4.3.4.gem
gem 'mysql2 (~> 0.5.0)' https://rubygems.org/downloads/mysql2-0.5.2.gem
gem 'yard' https://rubygems.org/downloads/yard-0.9.20.gem
gem 'mocha' https://rubygems.org/downloads/mocha-1.9.0.gem
gem 'metaclass' (~> 0.0.1) https://rubygems.org/downloads/metaclass-0.0.4.gem
gem 'simplecov (~> 0.14.1)' https://rubygems.org/downloads/simplecov-0.14.1.gem
gem 'docile' (~> 1.1.0) https://rubygems.org/downloads/docile-1.1.5.gem
gem 'simplecov-html' (~> 0.10.0) https://rubygems.org/downloads/simplecov-html-0.10.2.gem
gem 'puma (~> 3.7)' https://rubygems.org/downloads/puma-3.7.1.gem
gem 'capybara (~> 2.13)' https://rubygems.org/downloads/capybara-2.13.0.gem
gem 'xpath' (~> 2.0) https://rubygems.org/downloads/xpath-2.0.0.gem
gem 'mime-types' (>= 1.16) https://rubygems.org/downloads/mime-types-3.3.gem
gem 'mime-types-data' (~> 3.2015) https://rubygems.org/downloads/mime-types-data-3.2015.1120.gem
gem 'selenium-webdriver' https://rubygems.org/downloads/selenium-webdriver-3.142.6.gem
gem 'rubyzip' (>= 1.2.2) https://rubygems.org/downloads/rubyzip-1.3.0.gem
gem 'childprocess' (>= 0.5, < 4.0) https://rubygems.org/downloads/childprocess-3.0.0.gem
gem 'rake (>= 0.8.7)' https://rubygems.org/downloads/rake-13.0.1.gem
```

Положить их в каталог `/opt/redmine/redmine-4.0.5/vendor/cache`

И установить `bundler` и `passenger` локально

```sh
$ cd /opt/redmine/redmine-4.0.5/vendor/cache
$ sudo gem install bundler --local
$ sudo gem install passenger --local
```

## Установка Nginx + Passenger

Скачиваем Nginx и устанавливаем его с поддержкой Passenger

```sh
$ cd /tmp
$ curl -L https://nginx.org/download/nginx-1.17.5.tar.gz -o /tmp/nginx-1.17.5.tar.gz
$ tar zxf /tmp/nginx-1.17.5.tar.gz
$ sudo passenger-install-nginx-module --prefix=/opt/nginx --nginx-source-dir=/tmp/nginx-1.17.5 --languages ruby
```

```
Enter your choice (1 or 2) or press Ctrl-C to abort: 2 (собираем из исходников)
Please specify the directory: /tmp/nginx-1.17.5
Please specify a prefix directory [/opt/nginx]:

--------------------------------------------

Nginx with Passenger support was successfully installed.

The Nginx configuration file (/opt/nginx/conf/nginx.conf)
must contain the correct configuration options in order for Phusion Passenger
to function correctly.

This installer has already modified the configuration file for you! The
following configuration snippet was inserted:

  http {
      ...
      passenger_root /usr/local/lib/ruby/gems/2.6.0/gems/passenger-6.0.4;
      passenger_ruby /usr/local/bin/ruby;
      ...
  }

After you start Nginx, you are ready to deploy any number of Ruby on Rails
applications on Nginx.

Press ENTER to continue.

-------------------------------------------------
```

Для удобства создаём сим линк

```sh
$ sudo ln -s /opt/nginx/conf/ /etc/nginx
```

Редактируем файл конфигурации Nginx:

```sh
$ sudo nano /opt/nginx/conf/nginx.conf
user redmine;
worker_processes  auto;
...
```

В блок http добавляем следующий текст (в самом низу)

```sh
...
include sites-enabled/*.conf;
server_names_hash_bucket_size 64;
}
```

Далее настраиваем доступ к хосту Redmine

```sh
$ sudo mkdir /opt/nginx/conf/{sites-available,sites-enabled}
$ sudo nano /opt/nginx/conf/sites-available/redmine.conf
```

Пример конфигурационного файла для nginx

```sh
$ cat redmine.conf
server {
    listen 80 default_server;
    server_name redmine.example.com;

    gzip on;
    gzip_comp_level 7;
    gzip_types application/x-javascript application/javascript text/css;

    set $root_path /opt/redmine/redmine-latest/public;
    root $root_path;

    passenger_enabled on;
    passenger_document_root $root_path;
    client_max_body_size      100m; # Max attachemnt size
    client_body_buffer_size 4M;

    # redirect server error pages to the static page /50x.html
    error_page   500 502 503 504  /50x.html;

    location = /50x.html {
        root   html;
    }
		
    location ~* \.(jpg|gif|png|js|css|ico)$ {
        root $root_path;
        expires 7d;
    }

}
```

Делаем сим линк

```sh
$ cd /opt/nginx/conf/sites-available
$ sudo ln -s /opt/nginx/conf/sites-available/redmine.conf /opt/nginx/conf/sites-enabled/redmine.conf
```

Проверяем nginx

```sh
$ sudo /opt/nginx/sbin/nginx -t
```

Создаем файл для запуска nginx в качестве сервиса

```sh
$ sudo nano /lib/systemd/system/nginx.service
[Unit]
Description=The NGINX HTTP and reverse proxy server
After=syslog.target network.target remote-fs.target nss-lookup.target
 
[Service]
Type=forking
PIDFile=/opt/nginx/logs/nginx.pid
ExecStartPre=/opt/nginx/sbin/nginx -t
ExecStart=/opt/nginx/sbin/nginx
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s QUIT $MAINPID
PrivateTmp=true
 
[Install]
WantedBy=multi-user.target
```

Перечитываем конфигурации systemd

```sh
$ sudo systemctl daemon-reload
```

Запускаем Nginx и добавляем его в автозагрузку

```sh
$ sudo systemctl start nginx
$ sudo systemctl enable nginx
```

## Устанавливаем Redmine. Продолжение

Переходим в каталог

```sh
$ cd /opt/redmine/redmine-4.0.5/
```

Прописываем настройки для PostgreSQL

```sh
$ bundle config build.pg --with-pg-config=/usr/pgsql-10/bin/pg_config
```

Устанавливаем необходимые gems (локальная установка)

```sh
$ bundle install --without development test mysql2 sqlite --local
```

Либо, если у сервера есть выход в интернет, устанавливаем необходимые gems

```sh
$ bundle install --without development test mysql2 sqlite
```

Запускаем генерацию токена:

```sh
$ bundle exec rake generate_secret_token
```

Создаем структуру БД

```sh
$ RAILS_ENV=production bundle exec rake db:migrate
```

Загружаем в базу дефолтные данные

```sh
$ RAILS_ENV=production bundle exec rake redmine:load_default_data
  Select language: ru
```

Установка приложения Redmine завершена. Меняем владельца и прав доступа к каталогам и файлам

```sh
$ mkdir -p tmp tmp/pdf public/plugin_assets
$ chown -R redmine:redmine files log tmp public/plugin_assets
$ chmod -R 755 files log tmp public/plugin_assets
```

Перезагружаем nginx

```sh
$ sudo systemctl restart nginx
```

Осталось поменять пароль админа, для этого открываем браузер, переходим на соответствующую страницу и меняем пароль

## Настройка LDAP AD / FreeIPA

Настройка параметров LDAP производится через web-интерфейс

Пример параметров для подключения LDAP Active Directory

```
Имя: example.com
Компьютер: 192.168.0.2
Порт: 389 LDAP
Учётная запись: user@example.com
Пароль: •••••••••••••••
BaseDN: dc=example,dc=com
Создание пользователя на лету
Атрибут Login: sAMAccountName
Имя: givenName
Фамилия: sn
email: mail
```

Пример параметров для подключения LDAP FreeIPA

```
Имя: FreeIPA
Компьютер: freeipa.example.com
Порт: 636 LDAPS без проверки сертификата
Учётная запись
Пароль
BaseDN: cn=accounts,dc=example,dc=com
Создание пользователя на лету
Атрибут Login: uid
Имя: givenName
Фамилия: sn
email: mail
```
