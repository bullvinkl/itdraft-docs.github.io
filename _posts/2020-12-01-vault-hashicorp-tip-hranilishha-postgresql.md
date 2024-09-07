---
title: "HashiCorp Vault. Тип хранилища PostgreSQL"
date: "2020-12-01"
categories: 
  - Security-System
  - Database-System
tags: 
  - "hashicorp-vault"
  - "postgresql"
  - "journalctl"
  - "pgpass"
  - "pg_dump"
image:
  path: /commons/1.png
  alt: "HashiCorp Vault"
---

> **HashiCorp Vault** — это утилита командной строки, которая отвечает за управление секретами — логинами, паролями, ключами, сертификатами. «Управление» включает в себя как хранение, так и выдачу секретов конкретным приложениям с пометкой у себя в журнале, кому и когда это произошло.
{: .prompt-tip }

В прошлых статьях мы рассмотрели [установку HashiCorp Vault]({% post_url 2020-11-27-ustanovka-hashicorp-vault-v-centos-8 %}) и [установку PostgreSQL 13]({% post_url 2020-11-30-ustanovka-postgresql-13-v-centos-8 %}) в Centos 8.

## Настройки PostgreSQL

Создаем базу и пользователя. Для этого переключимся на пользователя postgres

```sh
$ sudo su - postgres
```

Создаем пользователя БД

```sh
$ createuser vltusr
```

Переключаемся в PostgreSQL shell

```sh
$ psql
```

Задаем пароль для пользователя БД

```sh
=# ALTER USER vltusr WITH ENCRYPTED password 'mypasswddd';
ALTER ROLE
```

Создам базу и задаем владельца базы

```sh
=# CREATE DATABASE vaultdb WITH ENCODING='UTF8' OWNER=vltusr;
CREATE DATABASE
```

Переключаемся на базу vaultdb

```sh
=# \c vaultdb
You are now connected to database "vaultdb" as user "postgres".
```

Создаем таблицу

```sh
=# CREATE TABLE vault_kv_store (
   parent_path TEXT COLLATE "C" NOT NULL,
   path        TEXT COLLATE "C",
   key         TEXT COLLATE "C",
   value       BYTEA,
   CONSTRAINT pkey PRIMARY KEY (path, key)
);
```

Создаем индекс

```sh
=# CREATE INDEX parent_path_idx ON vault_kv_store (parent_path);
```

Выходим

```sh
=# \q
$ exit
```

Тестируем подключение

```sh
$ sudo su - postgres
$ psql -U vltusr -h localhost -p 5432 vaultdb
Password for user vltusr: mypasswddd
=# \q
$ exit
```

## Настройка Vault

Останавливаем Vault

```sh
$ sudo systemctl stop vault
```

Редактируем конфиг Vault

```sh
$ sudo nano /etc/vault.d/vault.hcl
[…]
#storage "file" {
#  path  = "/var/lib/vault/data"
#}
storage "postgresql" {
  connection_url = "postgres://vltusr:mypasswddd@localhost:5432/vaultdb?sslmode=disable"
  table          = "vault_kv_store"
  max_parallel   = "128"
}
[…]
```

Переводим SELinux в режим работы `premissive`, иначе сервис Vault не будет стартовать

```sh
$ sudo setenforce 0
$ sudo nano /etc/selinux/config
[…]
SELINUX=permissive
```

Запускаем Vault

```sh
$ sudo systemctl start vault
```

Проверяем статус

```sh
$ systemctl status vault
$ journalctl -u vault
```

Дальше все манипуляции как в статье [по установке HashiCorp Vault в Centos 8]({% post_url 2020-11-27-ustanovka-hashicorp-vault-v-centos-8 %}):

- Добавление переменных
- Инициализация сервиса, где там выдадут токены
- Распечатывание хранилища
- Авторизация (vault login)

## Бэкапирование PostgreSQL

Создаем файл с параметрами подключения к базе, для того что бы в дальнейшем создавать дамп PostgreSQL без ввода пароля

```sh
$ sudo nano /root/.pgpass
# hostname:port:database:username:password
localhost:5432:vaultdb:vltusr:mypasswddd
```

Выставляем права

```sh
$ sudo chmod 600 /root/.pgpass
```

Создаем дамп

```sh
$ sudo pg_dump -d "vaultdb" -h localhost -Fc -U vltusr -w -f "/mnt/$(date +%Y%m%d_%H%M%S)_vaultdb.dump"
```
