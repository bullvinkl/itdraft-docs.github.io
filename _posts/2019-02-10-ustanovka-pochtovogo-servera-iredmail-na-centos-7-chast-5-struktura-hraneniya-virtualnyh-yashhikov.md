---
title: "Установка почтового сервера iRedMail на CentOS 7. Часть 5. Структура хранения виртуальных ящиков"
date: "2019-02-10"
categories: 
  - Linux
  - iRedMail
tags: 
  - "centos"
  - "iredmail"
image:
  path: /commons/DevOps-for-Network-Engineers.png
  alt: "Установка iRedMail"
---

> **iRedMail** - это бесплатное почтовое решение на базе открытого ПО, предназначенное для дистрибутивов Linux. Это полноценный почтовый сервер, который включает в себя Postfix и Dovecot, а также ряд дополнительных служб и настроек, разработанных для упрощения администрирования и обеспечения безопасности.
{: .prompt-tip }

Изначально почта хранится в каталоге `/var/vmail/vmail1/example.com`

Дальнейшая структура каталога имеет немного странный вид, например для ящика postmaster@example.com: `/var/vmail/vmail1/example.com/p/o/s/postmaster/`

Что бы изменить структуру, надо отредактировать файл `default_settings.py`

```sh
$ sudo nano /opt/iredadmin/libs/default_settings.py
MAILDIR_HASHED = False
MAILDIR_APPEND_TIMESTAMP = False
```

Новые почтовые ящики будут располагаться в каталоге `/var/vmail/vmail1/example.com`

Что бы изменить путь уже созданных ящиков надо отредактировать путь в базе `vmail`, таблица - `mailbox`, столбец - `maildir`  
И перенести каталог, на примере postmaster

```sh
$ sudo mv /var/vmail/vmail1/example.com/p/o/s/postmaster/ /var/vmail/vmail1/example.com/postmaster/
```
