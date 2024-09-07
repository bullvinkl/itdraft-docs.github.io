---
title: "Установка и настройка Memcached в CentOS 7"
date: "2019-08-14"
categories: 
  - Manuals
tags: 
  - "centos"
  - "memcached"
image:
  path: /commons/bigstock-laptop.jpg
  alt: "Установка и настройка Memcached"
---

> **Memcached** — программное обеспечение, реализующее сервис кэширования данных в оперативной памяти на основе хеш-таблицы. С помощью клиентской библиотеки позволяет кэшировать данные в оперативной памяти множества доступных серверов.
{: .prompt-tip }

## Устанавливаем сервис memcached

```sh
$ sudo yum -y install memcached
```

Запускаем сервис и добавляем его в автозагрузку

```sh
$ sudo systemctl start memcached
$ sudo systemctl enable memcached
```

## Настройка Memcached в режиме работы TCP

Для этого отредактируем конфигурационный файл

```sh
$ sudo nano /etc/sysconfig/memcached
USER="memcached"
PORT="11211"
MAXCONN="1024"
CACHESIZE="1024"
OPTIONS="-t 8 -l 127.0.0.1 -U 0"
```

где:  
- `MAXCONN = "1024"` - количество одновременных подключений (по умолчанию 1024);  
- `CACHESIZE="1024"` - объем выделяемой памяти для кеша (по умолчанию 64MB);  
- `OPTIONS="-t 8 -l 127.0.0.1 -U 0"` - количество потоков memcached 8(по умолчанию 4), прослушивать только localhost и отключим протокол UDP

Перезапустим Memcached

```sh
$ sudo systemctl restart memcached
```

Проверим, что Memcached привязан к локальному интерфейсу и прослушивает только TCP-соединения

```sh
$ sudo netstat -nltup
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
. . .
tcp        0      0 127.0.0.1:11211         0.0.0.0:*               LISTEN      2383/memcached
. . .
```

## Настройка Memcached в режиме работы SOCK

Отредактируем конфигурационный файл

```sh
$ sudo nano /etc/sysconfig/memcached
USER="memcached"
MAXCONN="1024"
CACHESIZE="1024"
OPTIONS="-t 8 -s /tmp/memcached.sock"
```

где  
- `USER="memcached"` - пользователь, от которого будет запущен memcached;  
- `OPTIONS="-t 8 -s /tmp/memcached.sock"` - количество потоков и путь к сокету.

Перезапустим Memcached

```sh
$ sudo systemctl restart memcached
```
