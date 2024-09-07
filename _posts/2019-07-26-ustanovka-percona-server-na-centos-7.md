---
title: "Установка Percona Server на Centos 7"
date: "2019-07-26"
categories: 
  - Linux
  - MySQL
tags: 
  - "centos"
  - "mysql"
  - "percona"
image:
  path: /commons/964732_881f.jpg
  alt: "Установка Percona Server"
---

> **Percona server** — это сборка MySQL с включенным по умолчанию XtraDB storage engine. Отличается от MySQL+InnoDB plugin лучшей производительностью/масштабируемостью, особенно на современных многоядерных серверах.
{: .prompt-tip }

## Установка Percona Server

Добавим репозиторий Percona и обновимся

```sh
$ sudo yum install https://repo.percona.com/yum/percona-release-latest.noarch.rpm
$ sudo yum update
```

Устанавливаем Percona-Server

```sh
$ sudo yum install Percona-Server-server-57
```

Запускаем сервис и добавляем его в автозагрузку

```sh
$ sudo systemctl start mysqld
$ sudo systemctl enable mysqld
```

Смотрим наш пароль, который сгенерировался автоматически

```sh
$ grep -i password /var/log/mysqld.log
2019-07-23T08:18:31.597730Z 1 [Note] A temporary password is generated for root@localhost: %password%
```

Меняем его. Для этого запускаем скрипт, задаем новый пароль, отвечаем на вопросы.

```sh
$ sudo mysql_secure_installation
```

## Создаем пользователя в базе

Подключаемся к mysql-серверу

```sh
$ mysql -u root -p
```

Создаем базу данных `db_name`

```sh
> CREATE DATABASE db_name;
```

Создаем пользователя `userdb_test` и назначаем ему пароль

```sh
> CREATE USER 'userdb_test'@'localhost' IDENTIFIED BY 'Bv&ea5dfgvR72';
```

Назначаем привилегии на базу

```sh
> GRANT ALL PRIVILEGES ON db_name.* TO 'userdb_test'@'localhost';
```

Обновляем привилегии

```sh
> FLUSH PRIVILEGES;
```

Выходим

```sh
> exit;
```
