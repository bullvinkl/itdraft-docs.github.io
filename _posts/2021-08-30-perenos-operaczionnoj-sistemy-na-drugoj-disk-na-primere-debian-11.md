---
title: "Перенос операционной системы на другой диск, на примере Debian 11"
date: "2021-08-30"
categories: 
  - Linux
  - Debian
tags: 
  - dd
  - debian
  - growpart
  - parted
  - mbr
  - gpt
  - xfs
  - ext4
image:
  path: /commons/156398bb67a3d4ee7ab97075a1137791.jpg
  alt: "Перенос операционной системы на другой диск"
---

## Клонирование системного диска

Клонирование системного диска будет осуществляться с помощью утилиты `DD`

Для начала устанавливаем утилиту `parted`

```sh
$ sudo apt install parted -y
```

Командой `fdisk` смотрим тип таблицы разделов на текущем диске (MBR или GPT)

```sh
$ sudo fdisk /dev/sda -l
Disk /dev/sda: 12.16 GiB, 13053992960 bytes, 25496080 sectors
Disk model: VBOX HARDDISK
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 2FD8E7D2-E382-43C0-9D78-8877EBCBBC2B
```

Подключаем новый диск, перезагружаемся.

Командой `parted` создаем новую таблицу разделов.

```sh
$ sudo parted /dev/sdb
```

Для EFI / GPT

```
> mklabel gpt
> quit
```

Для BIOS / MBR

```
> mklabel msdos
> quit
```

Командой `DD` клонируем `/dev/sda` в `/dev/sdb`

```sh
$ sudo dd if=/dev/sda of=/dev/sdb bs=1M conv=noerror,sync
```

Выключаем ВМ, отсоединяем старый диск, грузимся с нового

Если диски одного размера, на этом процесс завершен.  
Если новый диск большего размера, расширяем его.

## Увеличиваем корневой раздел

Для увеличения раздела нам понадобится утилита `growpart`, по умолчанию она не установлена. Ставим ее.

```sh
$ sudo apt install -y cloud-guest-utils
```

Синтаксис утилиты growpart:

```
growpart <device> <partition>
```

Расширяем раздел `3` на диске `/dev/sda`

```sh
$ sudo growpart /dev/sda 3
CHANGED: partition=3 start=503808 old: size=16271360 end=16775168 new: size=24992239 end=25496047
```

Расширяем физический том (physical volume)

```sh
$ sudo pvresize /dev/sda3
  Physical volume "/dev/sda3" changed
  1 physical volume(s) resized or updated / 0 physical volume(s) not resized
```

Смотрим путь и тип файловой системы (в данном примере xfs)

```sh
$ df -hT | grep mapper
Filesystem               Type      Size  Used Avail Use% Mounted on
/dev/mapper/debian-root xfs       6.8G  1.3G  5.6G  19% /
```

Расширяем логический том (logical volume)

```sh
$ sudo lvextend -r -l +100%FREE /dev/mapper/debian-root
```

Расширяем файловую систему XFS

```sh
$ sudo xfs_growfs /
```

Либо, расширяем файловую систему EXT4

```sh
$ sudo resize2fs /dev/mapper/centos-root
```