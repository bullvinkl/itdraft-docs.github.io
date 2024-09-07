---
title: "Локальный YUM репозиторий в Centos 7"
date: "2019-12-12"
categories: 
  - Manuals
tags: 
  - "centos"
  - "nginx"
  - "repository"
  - "rsync"
  - "yum"
image:
  path: /commons/0_HICLyAdNSIyT0ODU-1.jpg
  alt: "Локальный YUM репозиторий"
---

> **Репозиторий** — место, где хранятся и поддерживаются какие-либо данные. Чаще всего данные в репозитории хранятся в виде файлов, доступных для дальнейшего распространения по сети.  
> Среди дистрибутивов Linux популярны репозитории с форматом метаданных YUM для дистрибутивов на базе RPM-пакетов, и репозитории с метаданными APT для дистрибутивов на основе DEB-пакетов.
{: .prompt-tip }

Устанавливаем софт

```sh
$ sudo yum install createrepo yum-utils
```

Создаем каталоги os, updates, extras

```sh
$ mkdir -p /var/www/repo/centos/7/{os,updates,extras}/x86_64
```

Для синхронизации будем использовать зеркало Яндекса, т.к. у них заявлена поддержка rsync (873 исходящий порт)

```sh
$ rsync -iavrt --delete --exclude='repo*' rsync://mirror.yandex.ru/centos/7/os/x86_64/ /var/www/repo/centos/7/os/x86_64/
$ rsync -iavrt --delete --exclude='repo*' rsync://mirror.yandex.ru/centos/7/updates/x86_64/ /var/www/repo/centos/7/updates/x86_64/
$ rsync -iavrt --delete --exclude='repo*' rsync://mirror.yandex.ru/centos/7/extras/x86_64/ /var/www/repo/centos/7/extras/x86_64/
```

Далее надо поднять вэб-сервер.  
Пример конфигурации Nginx:

```sh
$ sudo cat /etc/nginx/site-avaliable/repo.conf
server {
    listen 80 default_server;
    server_name _;
    root /var/www/repo;
    charset UTF-8;
    default_type text/plain;

    location / {
        autoindex_exact_size off;
        autoindex on;
    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root /usr/share/nginx/html;
    }
}
```

Создаем репозитории

```sh
$ createrepo -v /var/www/repo/centos/7/os/x86_64
$ createrepo -v /var/www/repo/centos/7/updates/x86_64
$ createrepo -v /var/www/repo/centos/7/extras/x86_64
```

## Локальный репозиторий EPEL

Для создания локального репозитория EPEL создаем каталог

```sh
$ mkdir -p /var/www/repo/centos/7/epel/x86_64
```

Синхронизируемся c `mirror.logol.ru`, т.к. у них заявлена поддержка rsync для зеркала EPEL

```sh
$ rsync -iavrt --delete --exclude='repo*' rsync://mirror.logol.ru/epel/7/x86_64/ /var/www/repo/centos/7/epel/x86_64/
```

Создаем репозиторий

```sh
$ createrepo -v /var/www/repo/centos/7/epel/x86_64
```

## Обновление репозиториев

Для обновления репозиториев надо выполнить синхронизацию с источником

```sh
$ rsync -iavrt --delete --exclude='repo*' rsync://mirror.yandex.ru/centos/7/os/x86_64/ /var/www/repo/centos/7/os/x86_64/
$ rsync -iavrt --delete --exclude='repo*' rsync://mirror.yandex.ru/centos/7/updates/x86_64/ /var/www/repo/centos/7/updates/x86_64/
$ rsync -iavrt --delete --exclude='repo*' rsync://mirror.yandex.ru/centos/7/extras/x86_64/ /var/www/repo/centos/7/extras/x86_64/
$ rsync -iavrt --delete --exclude='repo*' rsync://mirror.logol.ru/epel/7/x86_64/ /var/www/repo/centos/7/epel/x86_64/
```

И обновить служебную информацию

```sh
$ createrepo --update /var/www/repo/centos/7/os/x86_64
$ createrepo --update /var/www/repo/centos/7/updates/x86_64
$ createrepo --update /var/www/repo/centos/7/extras/x86_64
$ createrepo --update /var/www/repo/centos/7/epel/x86_64
```

Данный набор из 4-х репозиториев на диске занимает приблизительно 60 Gb

## Автоматическое обновление репозиториев

Для автоматического обновления репозиториев создадим скрипт

```sh
$ sudo nano /home/repos_update.sh
#!/bin/bash
PATH=/etc:/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin

# os
rsync -iavrt --delete --exclude='repo*' rsync://mirror.yandex.ru/centos/7/os/x86_64/ /var/www/repo/centos/7/os/x86_64/
createrepo --update /var/www/repo/centos/7/os/x86_64

# update
rsync -iavrt --delete --exclude='repo*' rsync://mirror.yandex.ru/centos/7/updates/x86_64/ /var/www/repo/centos/7/updates/x86_64/
createrepo --update /var/www/repo/centos/7/updates/x86_64

# extras
rsync -iavrt --delete --exclude='repo*' rsync://mirror.yandex.ru/centos/7/extras/x86_64/ /var/www/repo/centos/7/extras/x86_64/
createrepo --update /var/www/repo/centos/7/extras/x86_64

# epel
rsync -iavrt --delete --exclude='repo*' rsync://mirror.logol.ru/epel/7/x86_64/ /var/www/repo/centos/7/epel/x86_64/
createrepo --update /var/www/repo/centos/7/epel/x86_64
```

Делаем скрипт исполняемым

```sh
$ sudo chmod +x /home/repos_update.sh
```

И добавляем задание по обновлению в crontab

```sh
$ crontab -e
# ежедневно в час ночи
0 1 * * * /home/repos_update.sh
```

## Добавить локальные репозитории на клиентские ПК / сервера

Для начала надо отключить имеющиеся конфигурационные файлы репозиториев

```sh
$ find /etc/yum.repos.d -type f -exec sed -i "s/enabled=1/enabled=0/g" {} \;
```

Создаем файл с настройками для локальных репозиториев

```sh
$ sudo nano /etc/yum.repos.d/local.repo
[local]
name=Local Yum Repo
baseurl=http://192.168.1.9/centos/$releasever/os/$basearch/
enabled=1
gpgcheck=0
priority=1

[local-update]
name=Local Yum Repo for update packages
baseurl=http://192.168.1.9/centos/$releasever/updates/$basearch/
enabled=1
gpgcheck=0
priority=1

[local-extras]
name=Local Yum Repo for extras packages
baseurl=http://192.168.1.9/centos/$releasever/extras/$basearch/
enabled=1
gpgcheck=0
priority=1
```

Создаем файл с настройками для локального репозитория EPEL

```sh
$ sudo nano /etc/yum.repos.d/local-epel.repo
[local-epel]
name=Local Extra Packages for Enterprise Linux 7
baseurl=http://192.168.1.9/centos/$releasever/epel/$basearch/
enabled=1
gpgcheck=0
```

Теперь можно обновляться с локальных репозиториев

```sh
$ sudo yum update
```
