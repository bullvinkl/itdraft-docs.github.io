---
layout: post
title: "[Решено] Netbox – backup, restore, upgrade"
# description: ""
date: 2023-08-20 21:36 +0300
categories:
  - Linux
  - Netbox
tags:
  - netbox
  - backup
  - restore
  - linux
  - postgresql
image:
  path: /commons/penguins.png
  alt: "Настройка LDAP и OIDC авторизации в Harbor для KeyCloak"
---

> *Netbox* — веб приложение с открытым исходным кодом, разработанное для управления и документирования компьютерных сетей. Изначально Netbox придуман командой сетевых инженеров DigitalOcean специально для системных администраторов.
{: .prompt-tip }

## Buckup and Restore Netbox

Создаем дамп базы данных через `pg_dump`
```sh
$ pg_dump -Fc -v --host=localhost --username=netbox --dbname=netboxdb -f netboxdb.dump
Password: passwdnetbox
```

Удаляем базу, создаем чистую, назначаем привилегии
```sh
$ sudo su - postgres
$ psql
=# DROP DATABASE netboxdb WITH (FORCE);
=# CREATE DATABASE netboxdb OWNER netbox;
=# GRANT ALL PRIVILEGES ON DATABASE netboxdb TO netbox;
=# \q
$ exit
```

Восстанавливаем дамп базы данных через `pg_restore`
```sh
$ pg_restore -v --no-owner --host=localhost --username=netbox --dbname=netboxdb netboxdb.dump
Password: passwdnetbox
```

## Upgrade Netbox

В данном случае я восстанавливал дамп с боевого сервера `Netbox v3.4.2` на локальном тесте `Netbox v3.6.7`. По этому пришлось сделать downgrade Netbox. В каталоге `/opt` у меня уже лежали 2 версии Netbox
```sh
$ cd /opt
$ sudo mv netbox netbox-3.6.7
$ sudo mv netbox-3.4.2/ netbox
$ cd netbox
$ sudo ./upgrade.sh
$ sudo systemctl restart netbox netbox-rq
$ ss -nltup
```

Заходим на наш локальный Netbox, и проверяем, что он успешно запустился, при этом видим, что в нем данные с боевого сервера
```sh
$ cd /opt
$ sudo mv netbox netbox-3.4.2
$ sudo ln -s netbox-3.6.7 netbox
$ sudo chown -h netbox:netbox netbox
$ sudo cp /opt/netbox-3.4.2/netbox/netbox/configuration.py /opt/netbox-3.6.7/netbox/netbox/
$ sudo cp /opt/netbox-3.4.2/local_requirements.txt /opt/netbox-3.6.7/local_requirements.txt
$ sudo cp /opt/netbox-3.6.7/contrib/gunicorn.py /opt/netbox-3.6.7/gunicorn.py
$ cd /opt/netbox
$ sudo ./upgrade.sh
$ sudo systemctl restart netbox netbox-rq
$ ss -nltup
```

Заходим на наш локальный Netbox, проверяем, что Netbox успешно загрузился
