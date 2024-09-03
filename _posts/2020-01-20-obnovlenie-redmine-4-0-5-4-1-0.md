---
title: "Обновление Redmine 4.0.5 - 4.1.0"
date: "2020-01-20"
categories: 
  - Linux
  - Redmine
tags: 
  - "centos"
  - "nginx"
  - "postgresql"
  - "redmine"
  - "upgrade"
  - "pgpass"
coverImage: "910838_84d6.jpg"
image:
  path: /commons/computer.jpg
  alt: ""
---

> **Redmine** — открытое серверное веб-приложение для управления проектами и задачами (в том числе для отслеживания ошибок). Redmine написан на Ruby и представляет собой приложение на основе широко известного веб-фреймворка Ruby on Rails

В предыдущей статье мы рассмотрели установку [Redmine 4.0.5 + Nginx + PostgreSQL]({% post_url 2019-12-10-ustanovka-redmine-4-0-5-nginx-postgresql-v-centos-7 %})  

## Подготовка

Рассмотрим задачу по обновлению Redmine до версии 4.1.0. Для начала сделаем дамп базы данных:

Создаем файл с параметрами подключения к базе, для того что бы в дальнейшем создавать дамп PostgreSQL без ввода пароля

```sh
$ sudo nano /root/.pgpass
#host:port:db:user:pass
localhost:5432:redmine:redmine:%password%

$ sudo chmod 600 /root/.pgpass
```

Запускаем создание дампа:

```sh
$ sudo pg_dump -d "redmine" -h localhost -Fc -U redmine -w -f "/opt/redmine/$(date +%Y%m%d_%H%M%S)_redmine.dump"
```

Скачиваем и распаковываем Redmine 4.1.0

```sh
$ curl -L https://www.redmine.org/releases/redmine-4.1.0.tar.gz -o /tmp/redmine-4.1.0.tar.gz
$ cd /tmp
$ tar zxf /tmp/redmine-4.1.0.tar.gz
$ sudo mv /tmp/redmine-4.1.0 /opt/redmine-4.1.0
```

Удаляем старый сим линк

```sh
$ sudo rm -r /opt/redmine-latest
```

Создаем новый сим линк и меняем права

```sh
$ sudo ln -s /opt/redmine-4.1.0 /opt/redmine-latest
$ sudo chown -h redmine:redmine /opt/redmine-latest
```

Если сервер не имеет выхода в интернет, нам заранее надо скачать gem-файлы, и положить их в каталог `/opt/redmine-4.1.0/vendor/cache/`

```
actioncable-5.2.4.1.gem
actionmailer-5.2.4.1.gem
actionpack-5.2.4.1.gem
actionpack-xml_parser-2.0.1.gem
actionview-5.2.4.1.gem
activejob-5.2.4.1.gem
activemodel-5.2.4.1.gem
activerecord-5.2.4.1.gem
activestorage-5.2.4.1.gem
activesupport-5.2.4.1.gem
addressable-2.7.0.gem
arel-9.0.0.gem
ast-2.4.0.gem
builder-3.2.4.gem
bundler-2.1.4.gem
capybara-3.25.0.gem
childprocess-3.0.0.gem
concurrent-ruby-1.1.5.gem
crass-1.0.6.gem
css_parser-1.7.1.gem
csv-3.1.2.gem
docile-1.1.5.gem
erubi-1.9.0.gem
globalid-0.4.2.gem
htmlentities-4.3.4.gem
i18n-1.6.0.gem
jaro_winkler-1.5.4.gem
json-2.3.0.gem
loofah-2.4.0.gem
mail-2.7.1.gem
marcel-0.3.3.gem
method_source-0.9.2.gem
mimemagic-0.3.3.gem
minitest-5.14.0.gem
mini_magick-4.9.5.gem
mini_mime-1.0.2.gem
mini_portile2-2.4.0.gem
mocha-1.11.2.gem
mysql2-0.5.3.gem
net-ldap-0.16.2.gem
nio4r-2.5.2.gem
nokogiri-1.10.7.gem
parallel-1.10.0.gem
parser-2.7.0.2.gem
pg-1.1.4.gem
public_suffix-4.0.3.gem
puma-3.7.1.gem
rack-2.1.1.gem
rack-openid-1.4.2.gem
rack-test-1.1.0.gem
rails-5.2.4.1.gem
rails-dom-testing-2.0.3.gem
rails-html-sanitizer-1.3.0.gem
railties-5.2.4.1.gem
rainbow-3.0.0.gem
rake-13.0.1.gem
rbpdf-1.20.1.gem
rbpdf-font-1.19.1.gem
redcarpet-3.5.0.gem
regexp_parser-1.5.1.gem
request_store-1.4.1.gem
roadie-4.0.0.gem
roadie-rails-2.1.1.gem
rouge-3.12.0.gem
rubocop-0.76.0.gem
rubocop-performance-1.5.2.gem
rubocop-rails-2.3.2.gem
ruby-openid-2.9.2.gem
ruby-progressbar-1.7.5.gem
rubyzip-2.0.0.gem
selenium-webdriver-3.142.7.gem
simplecov-0.17.0.gem
simplecov-html-0.10.2.gem
sprockets-4.0.0.gem
sprockets-rails-3.2.1.gem
thor-1.0.1.gem
thread_safe-0.3.6.gem
tzinfo-1.2.6.gem
unicode-display_width-1.6.0.gem
websocket-driver-0.7.1.gem
websocket-extensions-0.1.4.gem
xpath-3.2.0.gem
yard-0.9.24.gem
```

Копируем настройки из предыдущей установленной версии Redmine

```sh
$ cp /opt/redmine-4.0.5/config/database.yml /opt/redmine-4.1.0/config/database.yml
$ cp /opt/redmine-4.0.5/config/configuration.yml /opt/redmine-4.1.0/config/configuration.yml
```

Устанавливаем `bundler`

```sh
$ cd /opt/redmine-4.1.0/vendor/cache
$ sudo gem install bundler --local
```

Параметр `--local` - локальная установка

Если в этом месте возникла ошибка `comand not found`, выполнить

```sh
$ alias sudo='sudo env PATH=$PATH'
```

## Устанавливаем Redmine

Запускаем установку необходимых gem (в данном случае локально)

```sh
$ cd /opt/redmine-4.1.0/
$ bundle install --without development test mysql2 sqlite --local
```

Запускаем генерацию токена

```sh
$ bundle exec rake generate_secret_token
```

Обновляем в БД структуру

```sh
$ RAILS_ENV=production bundle exec rake db:migrate
```

Обновление Redmine завершена. Меняем владельца и прав доступа к каталогам и файлам

```sh
$ mkdir -p tmp tmp/pdf public/plugin_assets
$ chown -R redmine:redmine files log tmp public/plugin_assets
$ chmod -R 755 files log tmp public/plugin_assets
```

Перезагружаем nginx

```sh
$ sudo systemctl restart nginx
```