---
title: "Обновление SeaFile с версии 7.1.5 до 8.0.5 в Centos"
date: "2021-06-01"
categories: 
  - Storage-System
tags: 
  - "centos"
  - "seafile"
  - "upgrade"
image:
  path: /commons/template-1599665_1280.png
  alt: "Обновление SeaFile"
---

> **SeaFile** — это кроссплатформенная система программного обеспечения для размещения файлов с открытым исходным кодом. Файлы хранятся на центральном сервере и могут быть синхронизированы с персональными компьютерами и мобильными устройствами через приложения.
{: .prompt-tip }

Устанавливаем новые библиотеки Python

Для CentOS 7

```sh
$ sudo yum install python3-devel mysql-devel gcc gcc-c++ -y
```
 
> Что бы избежать ошибку
> /bin/ld: cannot find -lmysqlclient
> делаем сим линк на библиотеку `libmysqlclient`
{: .prompt-warning }

```sh
$ sudo ln -s /usr/lib64/mysql/libmysqlclient.so.18 /usr/lib/libmysqlclient.so
```

Продолжаем установку библиотек Python

```sh
$ sudo pip3 install future
$ sudo pip3 install mysqlclient==2.0.1 sqlalchemy==1.4.3
```

Для CentOS 8

```sh
$ yum install python3-devel mysql-devel gcc gcc-c++ -y
$ sudo pip3 install future mysqlclient sqlalchemy==1.4.3
```

Останавливаем сервисы `seafile` и `seahub`

```sh
$ sudo systemctl stop seafile seahub
```

Переключаемся на пользователя `seafile`

```sh
$ sudo su - seafile
```

Скачиваем дистрибутив SeaFile 8.0.5 (финальный релиз на сегодняшний день)

```sh
$ curl -OL https://s3.eu-central-1.amazonaws.com/download.seadrive.org/seafile-server_8.0.5_x86-64.tar.gz
```

Распаковываем его и перемещаем архив в каталог `installed`

```sh
$ tar xzf seafile-server_8.0.5_x86-64.tar.gz
$ mv seafile-server_8.0.5_x86-64.tar.gz installed
```

Запускаем скрипт для обновления

```sh
$ cd seafile-server-8.0.5/upgrade/
$ ./upgrade_7.1_8.0.sh
```

Переключаемся на предыдущего пользователя (с правами sudo)

```sh
$ exit
```

Запускаем SeaFile 8.0.5 server

```sh
$ sudo systemctl start seafile seahub
```
