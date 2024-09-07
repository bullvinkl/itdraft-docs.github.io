---
title: "[РЕШЕНО] LDAP-авторизация в PostgreSQL для FreeIPA"
date: "2023-04-03"
categories: 
  - Linux
  - FreeIPA
  - PostgreSQL
tags: 
  - "freeipa"
  - "ldap"
  - "linux"
  - "postgresql"
image:
  path: /commons/nn_guide-min.png
  alt: "[РЕШЕНО] LDAP-авторизация в PostgreSQL для FreeIPA"
---

> **FreeIPA** (IPA) - это комплексное решение для централизованного управления безопасностью Linux-систем, идентификацией и аутентификацией. Оно позволяет создавать многоуровневую систему управления доступом, где руководители подразделений могут добавлять пользователей в группы и настраивать доступ к ресурсам.
{: .prompt-tip }

У нас развернута служба каталогов на FreeIPA. Требуется подключить LDAP-авторизацию в PostgreSQL для пользователей FreeIPA из определенной группы

Требования:

- Должна быть настроена сетевая связанность по портам `389/tcp` (LDAP) или `636/tcp` (LDAPS) между сервером с PostgreSQL и FreeIPA
- Во FreeIPA должна быть добавлена служебная учетная запись (`postgresql_s`)
- Во FreeIPA должна быть создана группа пользователей, которым разрешен доступ в PostgreSQL (`g-postgres`)
- Все необходимые пользователи должны быть добавлены в эту группу

![](/assets/img/posts/2023/04/03/image.png){: w="300" }
_Группа пользователей FreeIPA_

Редактируем файл `pg_hba.conf` и вставляем строку подключения

```sh
$ sudo nano /etc/postgresql/14/main/pg_hba.conf
...
# Database administrative login by Unix domain socket
host    all     all     0.0.0.0/0       ldap ldapserver=freeipa.itdraft.lan ldapbasedn="cn=users,cn=accounts,dc=itdraft,dc=lan" ldapbinddn="uid=postgresql_s,cn=users,cn=accounts,dc=itdraft,dc=lan" ldapbindpasswd="passwd" ldapsearchattribute=uid
...
```

либо следующую строку, если хотим фильтрацию по группам

```sh
host    all     all     0.0.0.0/0       ldap ldapserver=freeipa.itdraft.lan ldapbasedn="cn=users,cn=accounts,dc=itdraft,dc=lan" ldapbinddn="uid=postgresql_s,cn=users,cn=accounts,dc=itdraft,dc=lan" ldapbindpasswd="passwd" ldapsearchfilter="(&(objectClass=person)(uid=$username)(memberOf=cn=g-postgres,cn=groups,cn=accounts,dc=itdraft,dc=lan))" 
```

> Строка подключения **должна идти самой первой** записью, что б отрабатывала LDAP-авторизация.
> Одновременно **ldapsearchattribute** и **ldapsearchfilter** не работают, PostgreSQL не запустится.
{: .prompt-warning }

Перезапускаем PostgreSQL

```sh
$ sudo systemctl restart postgresql
```

Либо перечитываем конфиг `pg_hba.conf` (без перезагрузки PostgreSQL)

```sh
$ sudo -u postgres psql -c "SELECT pg_reload_conf();"
```

> Пользователь должен быть локально добавлен в PostgreSQL (можно без пароля)

```sh
$ sudo -u postgres createuser m.makarov
```

Проверяем

```sh
$ psql postgres -h 127.0.0.1 -U m.makarov -W
Password: 
psql (14.7 (Debian 14.7-1.pgdg110+1))
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, bits: 256, compression: off)
Type "help" for help.

postgres=> 
```
