---
title: "Тестирование жесткого диска на чтение/запись в Centos 7"
date: "2018-07-25"
categories: 
  - Linux
tags: 
  - "centos"
  - "dd"
  - "hdparm"
image:
  path: /commons/data_leak.jpg
  alt: "Тестирование жесткого диска в Linux"
---

> Утилита **hdparm** позволяет получать информацию о параметрах жестких дисков, устанавливать параметры кэша, спящего режима, управления питанием. hdparm - мощная утилита для настройки и тестирования параметров жестких дисков, но ее использование требует осторожности и понимания возможных последствий неправильной настройки.

## Проверка скорости чтения диска

Для проверки скорости чтения нам потребуется утилита `hdparm`, установим ее  

```sh
$ sudo yum -y install hdparm
```

Смотрим список разделов

```sh
$ sudo fdisk -l

Disk /dev/sda: 2000.4 GB, 2000398934016 bytes, 3907029168 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
Disk label type: dos
Disk identifier: 0x000d428d

Устр-во Загр     Начало       Конец       Блоки   Id  Система
/dev/sda1   *        2048      526335      262144   83  Linux
/dev/sda2          526336     4720639     2097152   82  Linux swap / Solaris
/dev/sda3         4720640  3907028991  1951154176   83  Linux
```

Выбираем раздел, например `/dev/sda3`, и проверяем скорость чтения

```sh
$ sudo hdparm -t /dev/sda3

/dev/sda3:
 Timing buffered disk reads: 566 MB in  3.00 seconds = 188.35 MB/sec
```

## Проверка скорости записи диска

Для проверки скорости записи воспользуемся стандартной утилитой `dd`
Создадим файл размером 1 Gb частями по 1 Mb, выполним этот тест несколько раз

```sh
$ sudo sync; dd if=/dev/zero of=/opt/tempfile bs=1M count=1024; sync
1024+0 записей получено
1024+0 записей отправлено
 скопировано 1073741824 байта (1,1 GB), 7,77424 c, 138 MB/c
$ sudo sync; dd if=/dev/zero of=/opt/tempfile bs=1M count=1024; sync
1024+0 записей получено
1024+0 записей отправлено
 скопировано 1073741824 байта (1,1 GB), 7,84737 c, 137 MB/c
$ sudo sync; dd if=/dev/zero of=/opt/tempfile bs=1M count=1024; sync
1024+0 записей получено
1024+0 записей отправлено
 скопировано 1073741824 байта (1,1 GB), 7,9069 c, 136 MB/c
$ sudo sync; dd if=/dev/zero of=/opt/tempfile bs=1M count=1024; sync
1024+0 записей получено
1024+0 записей отправлено
 скопировано 1073741824 байта (1,1 GB), 7,86588 c, 137 MB/c
```