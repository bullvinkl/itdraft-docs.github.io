---
title: "Расширяем диск с помощью утилиты parted в Linux"
date: "2021-10-02"
categories: 
  - Linux
  - Disk
tags: 
  - centos
  - debian
  - linux
  - parted
  - rocky-linux
  - lvm
image:
  path: /commons/computer3.jpg
  alt: "Расширяем диск с помощью утилиты parted"
---

> **Parted** — свободный редактор дисковых разделов, предназначенный для создания и удаления разделов. Утилита полезна для создания разделов для новых операционных систем, реорганизации использования места на жёстком диске, копирования информации между дисками и создания образов диска.

В одной из прошлых статей был рассмотрен способ расширения диска с помощью утилиты [growpart]({% post_url 2021-07-23-rasshiryaem-lvm-razdel-bez-sozdaniya-novogo-fizicheskogo-toma-physical-volume-v-centos-rocky-linux %})

Но, при увеличении диска в Debian 8 столкнулся с тем, что с помощью `growpart` не получается расширить диск, появляется ошибка:

```
FAILED: failed to resize
**** WARNING: Resize failed, attempting to revert ****
Re-reading the partition table ...
sfdisk: BLKRRPART: Device or resource busy
sfdisk: The command to re-read the partition table failed.
Run partprobe(8), kpartx(8) or reboot your system now,
before using mkfs
**** Appears to have gone OK ****
```

Устанавливаем необходимый софт

```sh
$ sudo apt update
$ sudo apt -y install lvm2 parted xfsprogs
```

Расширяем необходимый диск в гипервизоре, перезагружаемся. Приступаем к расширению раздела

```sh
$ sudo parted /dev/sdb
(parted) print
Fix/Ignore? Fix
(parted) resizepart 1
End?  [4295MB]? 6392MB
(parted) print
(parted) quit
```

В данном примере мы расширяем первый раз раздел дика /dev/sdb с 4 Gb до 6 Gb

> Значение `6392MB` получаем после выполнения команды `print` в строке `Disk /dev/sdb:`

Ну и далее я расширил физический том (physical volume), lvm-раздел:

```sh
$ sudo pvresize /dev/sdb1
$ lsblk
$ df -hT | grep mapper
$ sudo lvextend -r -l +100%FREE /dev/mapper/storage--vg-vol_backups
$ sudo xfs_growfs /mnt/storage
```