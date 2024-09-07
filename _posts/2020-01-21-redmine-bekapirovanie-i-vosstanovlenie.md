---
title: "Redmine: бэкапирование и восстановление"
date: "2020-01-21"
categories: 
  - Project Management
tags: 
  - "backup"
  - "redmine"
  - "restore"
  - "pgpass"
image:
  path: /commons/987847_cbc9_2.jpg
  alt: "бэкапирование и восстановление"
---

> **Redmine** — открытое серверное веб-приложение для управления проектами и задачами (в том числе для отслеживания ошибок). Redmine написан на Ruby и представляет собой приложение на основе широко известного веб-фреймворка Ruby on Rails
{: .prompt-tip }

## Бэкапирование Redmine

Создаем файл с параметрами подключения к базе, для того что бы в дальнейшем создавать дамп PostgreSQL без ввода пароля

```sh
$ sudo nano /root/.pgpass
#host:port:base:user:pass
localhost:5432:redmine:redmine:%password%

$ sudo chmod 600 /root/.pgpass
```

Делаем бэкап базы:

```sh
$ sudo pg_dump -d "redmine" -h localhost -Fc -U redmine -w -f "/opt/redmine/$(date +%Y%m%d_%H%M%S)_redmine.dump"
```

Делаем бэкап директории, где установлен Redmine

```sh
$ tar -czf /opt/redmine/$(date +%Y%m%d_%H%M%S)_redmine.tar.gz /opt/redmine-4.0.5
```

Эти действия можно оформить скриптом и добавить скрипт в crontab

## Восстановление Redmine

Рассмотрим вариант, когда надо восстановить Redmine на чистый сервер.  
Вначале нам надо [установить Redmine]({% post_url 2019-12-10-ustanovka-redmine-4-0-5-nginx-postgresql-v-centos-7 %}) в соответствии с одной из предыдущих статей

Но п.7 **"Устанавливаем Redmine. Продолжение"** немного изменится:

Переходим в каталог и прописываем настройки для PostgreSQL

```sh
$ cd /opt/redmine-4.0.5/
$ bundle config build.pg --with-pg-config=/usr/pgsql-10/bin/pg_config
```

Устанавливаем необходимые gems (в данном случае локально)

```sh
$ bundle install --without development test mysql2 sqlite --local
```

Создаем файл с параметрами подключения к базе, для того что бы в дальнейшем создавать дамп PostgreSQL без ввода пароля

```sh
$ sudo nano /root/.pgpass
localhost:5432:redmine:redmine:%password%
$ sudo chmod 600 /root/.pgpass
```

Остановим web-сервер Nginx, что бы не было подключений к базе

```sh
$ sudo systemctl stop nginx
```

Подключаемся к БД

```sh
$ sudo su - postgres
-bash-4.2$ psql
```

Если в PostgreSQL есть база redmine к примеру с тестовыми данными, удаляем ее и создаем чистую

```sh
postgres=# DROP DATABASE redmine;
postgres=# CREATE DATABASE redmine WITH ENCODING='UTF8' OWNER=redmine;
postgres=# \q
-bash-4.2$ exit
```

Восстанавливаем базу из дампа и запускаем Nginx:

```sh
$ sudo pg_restore -h localhost -U redmine -d "redmine" -v "/tmp/20200115_094958_redmine.dump"
$ sudo systemctl start nginx
```

Восстанавливаем файлы Redmine из архива, сохранив их в каталог `/opt/nginx-4.0.5`

Запускаем генерацию токена:

```sh
$ bundle exec rake generate_secret_token
```

Обновляем структуру БД

```sh
$ RAILS_ENV=production bundle exec rake db:migrate
```

Перезагружаем Nginx

```sh
$ sudo systemctl restart nginx
```

Готово, Redmine восстановлен
