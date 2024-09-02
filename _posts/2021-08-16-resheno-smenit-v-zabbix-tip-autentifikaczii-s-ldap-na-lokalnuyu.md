---
title: "[РЕШЕНО] Сменить в Zabbix тип аутентификации с LDAP на локальную"
date: "2021-08-16"
categories: 
  - Linux
  - Zabbix
tags: 
  - "ldap"
  - "mysql"
  - "postgresql"
  - "zabbix"
image:
  path: /commons/warning02.jpg
  alt: "Сменить в Zabbix тип аутентификации"
---

После настройки LDAP-аутентификации, прежде чем менять аутентификацию по умолчанию необходимо добавить LDAP-пользователей, т.к. локальная учетная запись администратора становится неактивной, если аутентификацию по умолчанию переключить с внутренней на LDAP

![](/assets/img/posts/2021/08/16/image.png){: w="300" }

Но если вы все же сменили тип аутентификации по умолчанию в Zabbix, и не можете попасть в админку, достаточно выполнить SQL-запрос в консоли.

Для PostgreSQL

```sh
$ sudo su - postgres
$ psql zabbix
=# update config set authentication_type='0' where configid='1';
```

Для MySQL / MariaDB

```sh
$ mysql -u root -p
> update db_zabbix.config set authentication_type='0' where configid='1';
```