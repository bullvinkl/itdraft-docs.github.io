---
layout: post
title: "Подсветка синтаксиса в редакторе Nano"
date: "2024-11-05"
categories:
  - Manuals
tags:
  - nano
image:
  path: /commons/artificial-4082314_960_720.jpg
  alt: "Подсветка синтаксиса в редакторе Nano"
---

> **Nano** - простой и интуитивно понятный текстовый редактор, предназначенный для работы в командной строке. Он предоставляет основные функции для редактирования текста, включая создание, редактирование и сохранение файлов.
{: .prompt-tip }

## Установка Nano

По умолчанию, текстовый редактор Nano не установлен в Linux дистрибутивы. Для установки воспользуемся командой

```bash
$ sudo apt install nano   # Debian-like
$ sudo dnf install nano   # RHEL-like
```

## Подключаем подсветку синтаксиса

Что бы посмотреть, какой синтаксис поддерживаемся, необходимо посмотреть содержимое каталога `/usr/share/nano`

```bash
$ ls /usr/share/nano
```

![Images](/assets/img/posts/2024/11/05/nano1.png){: w="300" }
_Поддержка синтаксиса Nano_

По умолчанию подсветка синтаксиса в Nano не включена. Для включении подсветки синтаксиса отредактируем файл `~/.nanorc` и добавим некоторые

```bash
$ nano ~/.nanorc
include /usr/share/nano/default.nanorc
include /usr/share/nano/sh.nanorc
include /usr/share/nano/python.nanorc
include /usr/share/nano/c.nanorc
include /usr/share/nano/html.nanorc
include /usr/share/nano/markdown.nanorc
include /usr/share/nano/php.nanorc
include /usr/share/nano/yaml.nanorc
include /usr/share/nano/xml.nanorc
```

## Подключаем или отключаем подсветку синтаксиса без редактирования `.nanorc`

Чтобы включить подсветку синтаксиса, не редактируя файл `~/.nanorc`, добавим ключ `-Y` со значением из списка `ls /usr/share/nano`

```bash
$ nano -Ysh script.sh
```

либо с пробелом

```bash
$ nano -Y sh script.sh
```

Чтобы отключить подсветку синтаксиста, не редактируя файл `~/.nanorc`, добавим ключ `-Y` со значением `none` при запуске редактора

```bash
$ nano -Ynone script.sh
```

либо с пробелом

```bash
$ nano -Y none script.sh
```
