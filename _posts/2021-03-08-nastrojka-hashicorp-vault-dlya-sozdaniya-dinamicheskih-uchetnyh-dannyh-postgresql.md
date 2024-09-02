---
title: "Настройка HashiCorp Vault для создания динамических учетных данных PostgreSQL"
date: "2021-03-08"
categories: 
  - Linux
  - HashiCorp-Vault
  - PostgreSQL
tags: 
  - "hashicorp-vault"
  - "postgresql"
image:
  path: /commons/artificial-4082314_960_720.jpg
  alt: "Настройка HashiCorp Vault для создания динамических учетных данных PostgreSQL"
---

> **HashiСorp Vault** — это инструмент с открытым исходным кодом, предназначенный для безопасного хранения секретов и конфиденциальных данных в динамических облачных средах. Он обеспечивает надежное шифрование данных, доступ на основе идентификации с помощью настраиваемых политик.
> 
> Vault предоставляет механизм секретов, который можно настроить для создания набора динамических учетных данных с жесткой областью действия и установленным TTL (временем жизни).

Смотрим статус Vault

```sh
$  vault status
Key             Value
--- -----
Seal Type       shamir
Initialized     true
Sealed          true     <-- хранилище запечатано
Total Shares    5
Threshold       3
Version         1.6.3
Storage Type    postgresql
Cluster Name    vault
Cluster ID      6991c997-0e90-e38b-2ecb-d8ab4a677fa4
HA Enabled      false
```

Распечатываем хранилище

```sh
$ vault operator unseal
Unseal Key (will be hidden):
$ vault operator unseal
Unseal Key (will be hidden):
$ vault operator unseal
Unseal Key (will be hidden):
```

Вводим 3 ключа, т.к. при установке Vault мы это указывали

Авторизуемся в Vault из консоли

```sh
$ vault login
```

Включаем хранилище секретов для базы данных

```sh
$ vault secrets enable -path=psql database
```

Указываем плагин и информацию о подключении

```sh
$ vault write psql/config/my-postgresql-database \
    plugin_name=postgresql-database-plugin \
    allowed_roles="developer-role" \
    connection_url="postgresql://{{username}}:{{password}}@localhost:5432/postgres?sslmode=disable" \
    username="postgres" \
    password="mysuperpasswd"
```

Можно посмотреть, что записалось

```sh
$ vault read psql/config/my-postgresql-database
Key                                   Value
--- -----
allowed_roles                         [developer-role]
connection_details                    map[connection_url:postgresql://{{username}}:{{password}}@localhost:5432/postgres?sslmode=disable username:postgres]
password_policy                       n/a
plugin_name                           postgresql-database-plugin
root_credentials_rotate_statements    []
```

Настраиваем роль, которая сопоставляет имя в Vault с SQL-запросом для создания учетных записей базы данных

```sh
$ vault write psql/roles/developer-role \
    db_name=my-postgresql-database \
    creation_statements="CREATE ROLE \"{{name}}\" WITH LOGIN PASSWORD '{{password}}' VALID UNTIL '{{expiration}}'; \
        GRANT SELECT ON ALL TABLES IN SCHEMA public TO \"{{name}}\";" \
    default_ttl="1h" \
    max_ttl="24h"
```

Пробуем создать новую учетную запись

```sh
$ vault read psql/creds/developer-role
Key                Value
--- -----
lease_id           psql/creds/developer-role/bUpCqnFUq8EXF2TYhX529EWB
lease_duration     1h
lease_renewable    true
password           ZOhXBsoZcb6uz5ACFge-
username           v-root-develope-N2QYxWSbvc5ZFduq6JyY-1615063136
```

Проверяем подключение к PostgreSQL через консольный клиент `psql`

```sh
$ psql -h 127.0.0.1 \
    -d postgres \
    -U v-root-develope-N2QYxWSbvc5ZFduq6JyY-1615063136 \
    -W
Password: ZOhXBsoZcb6uz5ACFge-
psql (13.2)
Type "help" for help.

postgres=>
```

Список созданных учетных записей можно посмотреть:

```sh
$ vault list sys/leases/lookup/psql/creds/developer-role
Keys
----
B8hooXbcQMuY3GqNoDaWDubV
P0Q3txgk1Fp0NyEBl6m8f521
bUpCqnFUq8EXF2TYhX529EWB
```

Отозвать учетную запись, указав ее ID:

```sh
$ vault lease revoke psql/creds/developer-role/bUpCqnFUq8EXF2TYhX529EWB
All revocation operations queued successfully!
```

После отзыва учетной записи проверяем подключение к PostgreSQL через консольный клиент `psql`

```sh
$ psql -h 127.0.0.1 \
    -d postgres \
    -U v-root-develope-N2QYxWSbvc5ZFduq6JyY-1615063136 \
    -W
Password: ZOhXBsoZcb6uz5ACFge-
FATAL: password authentication failed for user "v-root-develope-N2QYxWSbvc5ZFduq6JyY-1615063136"
```

Отозвать все учетные записи

```sh
$ vault lease revoke -prefix psql/creds/developer-role
All revocation operations queued successfully!
```