---
title: "Обновляем Redmine с версии 2.4 до версии 2.6"
date: "2016-03-01"
categories: 
  - Project-Management
tags: 
  - "redmine"
  - "upgrade"
image:
  path: /commons/nn_guide-min.png
  alt: "Обновляем redmine"
---

> **Redmine** - это открытое серверное веб-приложение для управления проектами и задачами (в том числе для отслеживания ошибок). Разработано на языке Ruby и использует веб-фреймворк Ruby on Rails. Это гибкое веб-приложение, которое включает в себя диаграммы Ганта, календарь, вики, форумы, настройку ролей и уведомления по электронной почте.
{: .prompt-tip }

Делаем резервное копирование redmine и базы mysql

```sh
$ sudo tar -cvzf /tmp/$(date +%y%m%d)_redmine_backup.tar.gz /var/www/vhosts/redmine
$ sudo mysqldump -u redmine -predmine redmine > /tmp/$(date +%y%m%d)_redmine_backup.sql
```

Чистим директорию (`/var/www/html/redmine`)

```sh
$ sudo rm -rf /var/www/html/redmine/*
```

Скачиваем релиз

```sh
$ wget http://www.redmine.org/releases/redmine-2.6.3.zip
```

Распаковываем

```sh
$ sudo unzip redmine-2.6.3.zip
```

и переносим файлы в нужную директорию (`/var/www/html/redmine`)

Устанавливаем gems

```sh
$ sudo bundle install --without development test
```

Выполните следующую команду от вашего нового Redmine корневом каталоге:

```sh
$ sudo bundle exec rake generate_secret_token
```

ОЧЕНЬ ВАЖНО: Не прописывать данные в `config/settings.yml`

Копируем и переименовываем файл настройки коннекта к базе

```sh
$ sudo cp database.yml.example database.yml
```

Правим настройки подключения к базе

```sh
$ sudo nano database.yml
```

Обновляем базу

```sh
$ sudo bundle exec rake db:migrate RAILS_ENV=production
```

Ставим плагины

```sh
$ sudo bundle exec rake redmine:plugins:migrate RAILS_ENV=production
```

Чистим кэш, сессии

```sh
$ sudo bundle exec rake tmp:cache:clear tmp:sessions:clear RAILS_ENV=production
```
