---
title: "Перенос файлового хранилища SeaFile со всеми данными на другой сервер"
date: "2018-08-21"
categories: 
  - Storage-System
tags: 
  - "backup"
  - "centos"
  - "restore"
  - "seafile"
image:
  path: /commons/virtualisationimage.png
  alt: "Перенос файлового хранилища SeaFile"
---

> **SeaFile** – это облачное хранилище файлов с открытым исходным кодом, аналог Dropbox. Но в отличии от Dropbox, файлы хранятся вашем личном сервере. Файлы могут быть синхронизированы с персональными компьютерами и мобильными устройствами через приложения. Так же функционал Seafile позволяет предоставлять доступ к файлам как внешним пользователям, так и внутренним (другим зарегистрированным пользователям вашего хранилища).
{: .prompt-tip }

Чтобы перенести SeaFile со всеми пользователями и данными на другой сервер, необходимо:

- На старом сервере сделать бэкап mysql-базы и каталога, где лежит SeaFile
- На новом сервере установить и настроить mysql-сервер и web-сервер
- Перенести бэкап со старого сервера на новый
- Развернуть бэкап на новом сервере

Создаем резервную копию SeaFile на старом сервере

```sh
$ mysqldump -u seafile -ppassword ccnet_db | gzip > /home/backup/ccnet_db_$(date +%y%m%d).sql.gz
$ mysqldump -u seafile -ppassword seafile_db | gzip > /home/backup/seafile_db_$(date +%y%m%d).sql.gz
$ mysqldump -u seafile -ppassword seahub_db | gzip > /home/backup/seahub_db_$(date +%y%m%d).sql.gz

$ tar -zcf /home/backup/backup_$(date +%y%m%d).tar.gz /home/seafile
```

Разворачиваем бэкап на новом сервере

```sh
$ gzip -d /home/backup/ccnet_db_$(date +%y%m%d).sql.gz
$ gzip -d /home/backup/seafile_db_$(date +%y%m%d).sql.gz
$ gzip -d /home/backup/seahub_db_$(date +%y%m%d).sql.gz
```

Подключаемся к MySQL, создаем новые базы и создаем пользователя

```sh
$ mysql -u root -p
> CREATE DATABASE ccnet_db;
> CREATE DATABASE seafile_db;
> CREATE DATABASE seahub_db;
> create user 'seafile'@'localhost' identified by 'password';
```

Назначаем и обновляем привилегии и выходим

```sh
> GRANT ALL PRIVILEGES ON ccnet_db.* TO 'seafile'@'localhost';
> GRANT ALL PRIVILEGES ON seafile_db.* TO 'seafile'@'localhost';
> GRANT ALL PRIVILEGES ON seahub_db.* TO 'seafile'@'localhost';
> FLUSH PRIVILEGES;
> exit;
```

Восстанавливаем базу из dump'а

```sh
$ mysql -u seafile -ppassword ccnet_db < /home/backup/ccnet_db_$(date +%y%m%d).sql
$ mysql -u seafile -ppassword seafile_db < /home/backup/seafile_db_$(date +%y%m%d).sql
$ mysql -u seafile -ppassword seahub_db < /home/backup/seahub_db_$(date +%y%m%d).sql
```

Распаковываем архив с данными

```sh
$ sudo tar -xvzf /home/backup/backup_$(date +%y%m%d).tar.gz
```
Переносим распакованный архив в рабочий каталог, например `/home/seafile`

Дальнейшие действия описаны в статье по установке [SeaFile]({% post_url 2018-08-20-ustanovka-fajlovogo-hranilishha-seafile-na-centos-7 %})
