---
title: "Решение проблемы: Знаки вопросов вместо текста"
date: "2011-08-30"
categories: 
  - Manuals
tags: 
  - "mysql"
image:
  path: /commons/computer.jpg
  alt: "Решение проблемы: Знаки вопросов вместо текста"
---

> **UTF-8** (Universal Character Set Transformation Format - 8-bit) - это кодировка символов, позволяющая представлять текст на русском языке и других языках, включая кириллицу, латиницу и другие алфавиты.
{: .prompt-tip }

Случилось следующее: после переноса сайта на другой хостинг вместо русских букв стали отображаться знаки вопроса, что-то вроде: ????????? ?? ? ??? ????? ?? ???? ?


Добавление в `.htaccess` строчки `AddDefaultCharset windows-1251` не принесло результатов.

Решение проблемы:

Открываем файл настроек mysql (`/etc/my.cnf` - для Linux, `/usr/local/etc/my.cnf` - для FreeBSD)

В раздел `[mysqld]` необходимо добавить следующее:

```sh
default-character-set=utf8
character-set-server=utf8
collation-server=utf8_general_ci
init-connect="SET NAMES utf8"
skip-character-set-client-handshake
```

Две последние строки принудительно устанавливают кодировку `utf8` для всех запросов.

В раздел `[mysqldump]` достаточно добавить только

```sh
default-character-set=utf8
```

После перезагружаем Mysql
