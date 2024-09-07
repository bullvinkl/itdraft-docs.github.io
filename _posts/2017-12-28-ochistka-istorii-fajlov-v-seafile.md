---
title: "Очистка истории файлов в SeaFile"
date: "2017-12-28"
categories:
  - Linux
  - SeaFile
tags: 
  - "centos"
  - "seafile"
image:
  path: /commons/1039062_e553_4.jpg
  alt: "Очистка истории файлов в SeaFile"
---

> **SeaFile** - это личное хранилище для хранения данных в стиле Dropbox. Это кроссплатформенная система программного обеспечения с открытым исходным кодом, позволяющая создавать личное, семейное или корпоративное файловое хранилище. SeaFile обеспечивает безопасное хранение файлов на центральном сервере, синхронизируя их с персональными компьютерами и мобильными устройствами через приложения.
{: .prompt-tip }

На сервере стал занимать много места сервис SeaFile, после анализа стало ясно, что в настройках стояла опция хранить историю файлов. Отключаем эту опцию через аккаунт администратора,

```
Управление системой - Настройки - Library
```

Надо снять галку с пункта `library history - Allow user to keep library history`

![Allow user to keep library history](/assets/img/posts/2017/12/28/2018-08-22_14-12_seafile.png)

## Процедура очистки

Проверяем, сколько места занимают директории пользователей

```sh
$ du -sh /home/seafile/seafile-data/storage/blocks/*
223M /home/seafile/seafile-data/storage/blocks/478f96e8-b9b6-4b81-a920-eb191b53c82a
93G /home/seafile/seafile-data/storage/blocks/64ed8e7e-ca6d-4f60-bddf-7f6a8ab6205f
1,8G /home/seafile/seafile-data/storage/blocks/6e23ee5a-da44-4633-8893-0d2ec6ac5f04
8,0K /home/seafile/seafile-data/storage/blocks/70892c64-1330-42f9-adae-c80c39d5da93
8,0K /home/seafile/seafile-data/storage/blocks/7ead4d2f-5548-4e74-aa63-fd6eb9dc3e51
168K /home/seafile/seafile-data/storage/blocks/99485677-daf2-4766-aa28-27353c5ab133
9,4G /home/seafile/seafile-data/storage/blocks/a84ee2db-1a1c-4811-acc1-35f771ff55dd
3,6G /home/seafile/seafile-data/storage/blocks/afb8315d-dcc9-415e-bbae-42ab13a06684
285M /home/seafile/seafile-data/storage/blocks/b26f305b-23cb-4753-90c3-fc45c946688a
877M /home/seafile/seafile-data/storage/blocks/b339f48e-7945-4ef0-b630-2e0ee2eb06eb
332M /home/seafile/seafile-data/storage/blocks/c605ae9f-8b27-4fd1-9f7c-f4f9363e6de0
302M /home/seafile/seafile-data/storage/blocks/c992c3a9-40b0-49e3-af0e-467ad39b717d
8,0K /home/seafile/seafile-data/storage/blocks/dd168831-208f-4d8e-beb0-39375f2287c8
124M /home/seafile/seafile-data/storage/blocks/e264d4ad-875f-4b0f-aa7d-63f853e862a7
39G /home/seafile/seafile-data/storage/blocks/f977530a-0975-46c8-825f-27461384271e
93G /home/seafile/seafile-data/storage/blocks/ff147da3-452b-493f-a0ca-167c52693fd3
```

где `ff147da3-452b-493f-a0ca-167c52693fd3` — id библиотеки

Как видно, у некоторых пользователей занято по 90 Gb, хотя в самом аккаунте хранится не более 2 Gb

Авторизуемся пользователем, заходим в «мою библиотеку», находим иконку корзины и очищаем корзину.  
Такую процедуру выполняем с каждым пользователем, у кого директория занимает много места.

Останавливаем сервис, смотрим что можно удалить у пользователя, очищаем

```sh
$ sudo service seafile stop
$ sudo cd /home/seafile/seafile-server-latest
$ sudo ./seaf-gc.sh --dry-run a84ee2db-1a1c-4811-acc1-35f771ff55dd
$ sudo ./seaf-gc.sh a84ee2db-1a1c-4811-acc1-35f771ff55dd
```

Эту процедуру повторяем для каждого пользователя

Проверяем

```sh
$ du -sh /home/seafile/seafile-data/storage/blocks/*
223M /home/seafile/seafile-data/storage/blocks/478f96e8-b9b6-4b81-a920-eb191b53c82a
6,2G /home/seafile/seafile-data/storage/blocks/64ed8e7e-ca6d-4f60-bddf-7f6a8ab6205f
1,8G /home/seafile/seafile-data/storage/blocks/6e23ee5a-da44-4633-8893-0d2ec6ac5f04
8,0K /home/seafile/seafile-data/storage/blocks/70892c64-1330-42f9-adae-c80c39d5da93
8,0K /home/seafile/seafile-data/storage/blocks/7ead4d2f-5548-4e74-aa63-fd6eb9dc3e51
168K /home/seafile/seafile-data/storage/blocks/99485677-daf2-4766-aa28-27353c5ab133
518M /home/seafile/seafile-data/storage/blocks/a84ee2db-1a1c-4811-acc1-35f771ff55dd
3,6G /home/seafile/seafile-data/storage/blocks/afb8315d-dcc9-415e-bbae-42ab13a06684
285M /home/seafile/seafile-data/storage/blocks/b26f305b-23cb-4753-90c3-fc45c946688a
877M /home/seafile/seafile-data/storage/blocks/b339f48e-7945-4ef0-b630-2e0ee2eb06eb
332M /home/seafile/seafile-data/storage/blocks/c605ae9f-8b27-4fd1-9f7c-f4f9363e6de0
302M /home/seafile/seafile-data/storage/blocks/c992c3a9-40b0-49e3-af0e-467ad39b717d
8,0K /home/seafile/seafile-data/storage/blocks/dd168831-208f-4d8e-beb0-39375f2287c8
124M /home/seafile/seafile-data/storage/blocks/e264d4ad-875f-4b0f-aa7d-63f853e862a7
7,6G /home/seafile/seafile-data/storage/blocks/f977530a-0975-46c8-825f-27461384271e
1,8G /home/seafile/seafile-data/storage/blocks/ff147da3-452b-493f-a0ca-167c52693fd3
```

Как видно, удалось освободить кучу места

До очистки Seafile

```sh
$ sudo df -h
Filesystem Size Used Avail Use% Mounted on
/dev/sda3 438G 338G 79G 82% /
```

После очистки Seafile

```sh
$ sudo df -h
Filesystem Size Used Avail Use% Mounted on
/dev/sda3 438G 125G 292G 30% /
```

И не забываем запустить сервис обратно

```sh
$ sudo service seafile start
```

## UPD 12.11.2020

Авторизуемся в вэб-интерфейсе админом, переходим в область для администрирования, далее:  
Библиотека > Корзина

Удаляем содержимое корзины

Останавливаем сервис seafile

```sh
$ sudo systemctl stop seafile
```

Переходим в каталог seafile-server-latest

```sh
$ cd /opt/seafile/seafile-server-latest
```

Удаляем с сервера все, что почистили в корзине

```sh
$ sudo ./seaf-gc.sh -r
```

Запускаем сервис

```sh
$ sudo systemctl start seafile
```
