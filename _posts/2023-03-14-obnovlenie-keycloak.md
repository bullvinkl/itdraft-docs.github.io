---
title: "Обновление Keycloak"
date: "2023-03-14"
categories: 
  - Linux
  - Keycloak
tags: 
  - "keycloak"
  - "linux"
  - "pg_dump"
  - "upgrade"
image:
  path: /commons/analytics.jpg
  alt: "Обновление Keycloak"
---

> **Keycloak** — продукт с открытым кодом для реализации single sign-on с возможностью управления доступом, нацелен на современные применения и сервисы. По состоянию на 2018 год, этот проект сообщества JBoss находится под управлением Red Hat которые используют его как upstream проект для своего продукта RH-SSO

- Список статей из категории [Keycloak](/categories/keycloak/)

На официальном сайте про обновление Keycloak сказано следующее:

- скачать и распаковать архив

- скопировать содержимое `conf/`, `providers/` и `themes/` в новое расположение

- пересобрать Keycloak (Re-build)

Обновление будем производить с версии 19.0.1 до 21.0.0. На момент написания статьи это была финальная версия

Скачиваем, распаковываем архив и назначаем права

```sh
$ sudo wget https://github.com/keycloak/keycloak/releases/download/21.0.0/keycloak-21.0.0.zip -P /opt/keycloak
$ sudo unzip /opt/keycloak/keycloak-21.0.0.zip -d /opt/keycloak
$ sudo chown -R keycloak. /opt/keycloak/keycloak-21.0.0
$ sudo chmod o+x /opt/keycloak/keycloak-21.0.0/bin/
```

Сохраняем дефолтеый конфиг

```sh
$ sudo mv /opt/keycloak/keycloak-21.0.0/conf/keycloak.conf /opt/keycloak/keycloak-21.0.0/conf/keycloak.conf-origin
```

Копируем ssl-сертификаты и конфигурационный файл с настройками

```sh
$ sudo cp /opt/keycloak/keycloak-19.0.1/conf/server* /opt/keycloak/keycloak-21.0.0/conf/
$ sudo cp /opt/keycloak/keycloak-19.0.1/conf/keycloak.conf /opt/keycloak/keycloak-21.0.0/conf/
```

Редактируем конфигурационный файл

```sh
$ sudo nano /opt/keycloak/keycloak-21.0.0/conf/keycloak.conf
...
# The file path to a server certificate or certificate chain in PEM format.
https-certificate-file=/opt/keycloak/keycloak-21.0.0/conf/server.crt.pem

# The file path to a private key in PEM format.
https-certificate-key-file=/opt/keycloak/keycloak-21.0.0/conf/server.key.pem
```

Останавливаем сервис

```sh
$ sudo systemctl stop keycloak
```

Пересобираем Keycloak

```sh
$ cd /opt/keycloak/keycloak-21.0.0
$ sudo bin/kc.sh build
```

На всякий случай делаем дамп нашей базы

```sh
$ pg_dump -h localhost -U keycloak -W keycloak > /tmp/keycloak_20230225.dump
```

Пробуем запустить

```sh
$ sudo bin/kc.sh start --hostname=keycloak.itdraft.ru
```

> Важно. В версии 21.0.0 был баг, Keycloak не хотел запускаться, появлялась ошибка:  
> ERROR \[org.keycloak.quarkus.runtime.cli.ExecutionExceptionHandler\]

Решение ошибки: обновляем параметр в базе

```sh
$ sudo su - postgres -c psql
=# \c keycloak
=# update realm set admin_theme = 'keycloak' where admin_theme is null;
```

Редактируем системный юнит

```sh
$ sudo nano /etc/systemd/system/keycloak.service
...
ExecStart=!/opt/keycloak/keycloak-21.0.0/bin/kc.sh start --hostname=keycloak.mydomain.com
...
```

Перечитываем изменения и запускаем сервис

```sh
$ sudo systemctl daemon-reload
$ sudo systemctl start keycloak
$ sudo systemctl status keycloak
```