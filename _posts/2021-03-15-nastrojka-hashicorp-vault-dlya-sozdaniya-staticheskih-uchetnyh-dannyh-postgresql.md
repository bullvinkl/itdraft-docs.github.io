---
title: "Настройка HashiCorp Vault для создания статических учетных данных PostgreSQL"
date: "2021-03-15"
categories: 
  - Security-System
  - Database-System
tags: 
  - "hashicorp-vault"
  - "postgresql"
image:
  path: /commons/evillimiter-featured.png
  alt: "Настройка HashiCorp Vault для создания статических учетных данных PostgreSQL"
---

> **HashiСorp Vault** — это инструмент с открытым исходным кодом, предназначенный для безопасного хранения секретов и конфиденциальных данных в динамических облачных средах. Он обеспечивает надежное шифрование данных, доступ на основе идентификации с помощью настраиваемых политик.
> Vault предоставляет механизм секретов, который можно настроить для создания набора статических учетных данных с жесткой областью действия и установленным TTL (временем жизни).
{: .prompt-tip }

Проверяем статус Vault

```sh
$ vault status
```

Распечатываем хранилище

```sh
$ vault operator unseal
```

Авторизуемся

```sh
$ vault login
Token (will be hidden):
```

## Настройка Vault

Включаем хранилище секретов для базы данных

```sh
$ vault secrets enable -path=postgresql database
```

Указываем какой плагин для Vault мы будем использовать, и информацию о подключении

```sh
$ vault write postgresql/config/connection \
    plugin_name=postgresql-database-plugin \
    allowed_roles="*" \
    connection_url=postgresql://{{username}}:{{password}}@localhost:5432/postgres?sslmode=disable \
    username="postgres" \
    password="mysuperpassword"
```

В данном примере СУБД PostgreSQL на одном сервере с Vault, отсюда и `localhost:5432`

Можно посмотреть, что записалось

```sh
$ vault read postgresql/config/connection
Key                                   Value
--- -----
allowed_roles                         [*]
connection_details                    map[connection_url:postgresql://{{username}}:{{password}}@localhost:5432/postgres?sslmode=disable username:postgres]
password_policy                       n/a
plugin_name                           postgresql-database-plugin
root_credentials_rotate_statements    []
```

Что бы сбросить пароль пользователя `postgres`, надо выполнить:

```sh
$ vault write -force postgresql/rotate-root/connection
Success! Data written to: postgresql/rotate-root/connection
```

Создаем роль в БД и назначаем привилегии (без этого следующий запрос vault не отработает)

```sh
$ sudo su - postgres
$ psql
# CREATE ROLE "vault-edu" WITH LOGIN PASSWORD 'mypassword';
# GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO "vault-edu";
# \q
$ exit
```

Теперь создадим статическую роль `education` в HashiCorp Vault

```sh
$ vault write postgresql/static-roles/education \
    db_name=connection \
    rotation_statements="ALTER USER \"{{name}}\" WITH PASSWORD '{{password}}';" \
    username="vault-edu" \
    rotation_period=86400
Success! Data written to: postgresql/static-roles/education
```

Приведенная выше команда создает статическую роль `education` с именем пользователя `vault-edu`, пароль которой меняется каждые 86400 секунд (24 часа).

Выполним следующую команду, чтобы прочитать определение роли `education`

```sh
$ vault read postgresql/static-roles/education
Key                    Value
--- -----
db_name                postgresql
last_vault_rotation    2019-06-24T10:18:39.766203-07:00
rotation_period        24h
rotation_statements    [ALTER USER "{{name}}" WITH PASSWORD '{{password}}';]
username               vault-edu
```

## Политика доступа Vault

Чтобы получить учетные данные для статической роли `vault-edu`, клиентское приложение должно иметь возможность читать из конечной точки роли postgresql / static-creds / education. Следовательно, токен приложения должен иметь политику, предоставляющую разрешение на чтение.

Создадим файл `apps.hcl` со следующим содержимым

```sh
$ nano apps.hcl
# Get credentials from the database secrets engine
path "postgresql/static-creds/education" {
  capabilities = [ "read" ]
}
```

Создадим политику `apps` в Vault

```sh
$ vault policy write apps apps.hcl
Success! Uploaded policy: apps
```

Создадим токен, чтобы мы могли пройти аутентификацию для чтения статических паролей

```sh
$ vault token create -policy="apps"
Key                  Value
--- -----
token                s.osFnGR3JAUMI00pmnebMG40g
token_accessor       8UWwnO12IMrN6gIIqsLZXuK5
token_duration       10h
token_renewable      true
token_policies       ["apps" "default"]
identity_policies    []
policies             ["apps" "default"]
```

Выполним следующую команду, чтобы запросить учетные данные для роли `vault-edu`. Обязательно используем токен, который получили в предыдущем шаге.

```sh
$ VAULT_TOKEN=s.osFnGR3JAUMI00pmnebMG40g vault read postgresql/static-creds/education
Key                    Value
--- -----
last_vault_rotation    2021-03-08T19:39:04.336858834+03:00
password               Wed-OfKj2pdWQnnYcBAe
rotation_period        24h
ttl                    23h52m38s
username               vault-edu
```

Проверяем через `psql`

```sh
$ psql -h 127.0.0.1 \
    -d postgres \
    -U vault-edu \
    -W
Password for user vault-edu: Wed-OfKj2pdWQnnYcBAe
```

```sh
postgres=> \du
                                   List of roles
 Role name |                         Attributes                         | Member
 of
-----------+------------------------------------------------------------+-------

 postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
 vault-edu |                                                            | {}
```

Пароль для статической роли автоматически меняется после заданного периода ротации. Однако может возникнуть ситуация, требующая немедленной смены пароля.

Выполним следующую команду, чтобы изменить пароль для статической роли `education`

```sh
$ vault write -f postgresql/rotate-role/education
Success! Data written to: postgresql/rotate-role/education
```

Теперь прочитаем учетные данные, чтобы убедиться, что пароль был изменен

```sh
$ vault read postgresql/static-creds/education
Key                    Value
--- -----
last_vault_rotation    2021-03-08T19:51:35.606636934+03:00
password               QYpe-8w1-Dc7kG0szQxZ
rotation_period        24h
ttl                    23h59m11s
username               vault-edu
```

Возвращенный пароль должен отличаться от предыдущего вывода, а оставшийся TTL вернулся к ~ 24 часам
