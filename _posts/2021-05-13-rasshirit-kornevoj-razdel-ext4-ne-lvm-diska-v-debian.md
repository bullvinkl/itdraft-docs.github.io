---
title: "Расширить корневой раздел (ext4, не LVM) диска в Debian"
date: "2021-05-13"
categories: 
  - Linux
  - Disk
tags: 
  - "cfdisk"
  - "debian"
  - "fdisk"
coverImage: "laptops.png"
image:
  path: /commons/laptops.png
  alt: "Расширить корневой раздел ext4"
---

> Корневой раздел является хранилищем всех остальных файловых систем. Через него система получает доступ ко многим (если не ко всем) своим ресурсам. В этом разделе (файловая система) содержит такие важные системные каталоги (которые могут быть выноситься в отдельные разделы при желании и являться отдельными файловыми системами) как «/usr», «/bin», «/etc», «/var», «/opt» и т. д., в совокупности все они содержат файлы ядра, стандартные системные утилиты, файлы хранимой конфигурации системы, файлы журналов системных событий и т. д.

Есть виртуальная машина, разбивка диска следующая:

```sh
$ lsblk
$ df -H
$ sudo cfdisk /dev/sda
```

![](/assets/img/posts/2021/05/13/ext01.png){: w="300" }
_lsblk, df -H_

![](/assets/img/posts/2021/05/13/ext02.png){: w="300" }
_cfdisk_

Выключаем виртуалку, увеличиваем размер vdi-диска с помощью VBoxManage (в составе VirtualBox):

```
"C:\Program Files\Oracle\VirtualBox\VBoxManage.exe" modifyhd "C:\Users\user\VirtualBox VMs\debian8\debian8.vdi" --resize 12288
```

Включаем виртуалку, смотрим что получилось

```sh
$ lsblk
$ df -H
$ sudo cfdisk /dev/sda
```

![](/assets/img/posts/2021/05/13/ext03.png){: w="300" }
_lsblk, df -H_

![](/assets/img/posts/2021/05/13/ext04.png){: w="300" }
_cfdisk_

Выключаем swap (файл подкачки)

```sh
$ sudo swapoff -a
```

Начинаем удалять разделы (данные не потеряются)

Смотрим разметку

```sh
$ sudo fdisk /dev/sda

Command (m for help): p
Disk /dev/sda: 12 GiB, 12821987328 bytes, 25042944 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xd00b3928

Device     Boot    Start      End  Sectors  Size Id Type
/dev/sda1  *        2048 15988735 15986688  7.6G 83 Linux
/dev/sda2       15990782 16775167   784386  383M  5 Extended
/dev/sda5       15990784 16775167   784384  383M 82 Linux swap / Solaris
```

В данном примере вначале удаляем /dev/sda2

```
Command (m for help): d
Partition number (1,2,5, default 5): 2

Partition 2 has been deleted.
```

Смотрим результат

```
Command (m for help): p
Disk /dev/sda: 12 GiB, 12821987328 bytes, 25042944 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xd00b3928

Device     Boot Start      End  Sectors  Size Id Type
/dev/sda1  *     2048 15988735 15986688  7.6G 83 Linux
```

Удаляем раздел /dev/sda1 (данные не потеряются)

```
Command (m for help): d
Selected partition 1
Partition 1 has been deleted.
```

Таким образом мы удалили разделы на диске. Данный способ используется т.к. при автоматической разбивке диска в Debian (без LVM) корневой раздел оказывается в начале диска, а добавляемое пространство оказывается в конце диска. А между ними область, выделенная под swap.

Создадим новый раздел (primary)

```
Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (1-4, default 1):
First sector (2048-25042943, default 2048):
Last sector, +sectors or +size{K,M,G,T,P} (2048-25042943, default 25042943): +11G

Created a new partition 1 of type 'Linux' and of size 11 GiB.
```

Таким образом мы создали новый раздел размером 11 Gb, 1 Gb оставили под swap

Создадим раздел (extended) под swap

```
Command (m for help): n
Partition type
   p   primary (1 primary, 0 extended, 3 free)
   e   extended (container for logical partitions)
Select (default p): e
Partition number (2-4, default 2):
First sector (23070720-25042943, default 23070720):
Last sector, +sectors or +size{K,M,G,T,P} (23070720-25042943, default 25042943):

Created a new partition 2 of type 'Extended' and of size 963 MiB.
```

Попробуем поменять тип файловой системы

```
Command (m for help): t
Partition number (1,2, default 2): 2
Hex code (type L to list all codes): 82
You cannot change a partition into an extended one or vice versa. Delete it first.

Type of partition 2 is unchanged: Extended.
```

Утилита ругается

Сохраняем изменения

```
Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Re-reading the partition table failed.: Device or resource busy

The kernel still uses the old table. The new table will be used at the next reboot or after you run partprobe(8) or kpartx(8).
```

Утилита сообщает, что изменения применятся после перезагрузки

Перезагружаем виртуалку

```sh
$ sudo reboot
```

Смотрим результат

```sh
$ lsblk
$ df -H
$ sudo cfdisk /dev/sda
```

![](/assets/img/posts/2021/05/13/ext06-1.png){: w="300" }
_lsblk, df -H_

![](/assets/img/posts/2021/05/13/ext07.png){: w="300" }
_cfdisk_

Запускаем утилиту cfdisk

```sh
$ sudo cfdisk /dev/sda
```

Выбираем /dev/sda1:

```
Bootable
```

Выбираем неразмеченную область:

```
New - > Partition size: 962M -> Type: 82
```

![](/assets/img/posts/2021/05/13/ext08.png){: w="300" }
_New_

Сохраняем изменения

```
Write: yes - > Quit
```

![](/assets/img/posts/2021/05/13/ext09.png){: w="300" }
_Write: yes_

Передаем информацию об изменении разметки операционной системе, установив утилиту parted

```sh
$ sudo apt install parted -y
$ sudo partprobe
```

Создаем раздел под swap

```sh
$ sudo mkswap /dev/sda5
Setting up swapspace version 1, size = 985084 KiB
no label, UUID=be9028ea-7dd0-445b-99ae-69835d586ed5
```

Включаем swap

```sh
$ sudo swapon /dev/sda5
```

Смотрим новые UUID

```sh
$ sudo blkid
/dev/sda1: UUID="c86485d2-6505-426e-9298-48eb1462be89" TYPE="ext4" PARTUUID="d00b3928-01"
/dev/sda5: UUID="be9028ea-7dd0-445b-99ae-69835d586ed5" TYPE="swap" PARTUUID="d00b3928-05"
```

Прописываем их в /etc/fstab

```sh
$ sudo nano /etc/fstab
```

Монтируем

```sh
$ sudo mount -a
```

Перезагружаем виртуалку

```sh
$ sudo reboot
```

Проверяем

```sh
$ lsblk
$ df -h
```

![](/assets/img/posts/2021/05/13/ext10.png){: w="300" }
_lsblk, df -h_

Расширяем раздел /dev/sda1

```sh
$ sudo resize2fs /dev/sda1
```

Проверяем

```sh
$ df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1        11G  1.2G  9.1G  12% /
udev             10M     0   10M   0% /dev
tmpfs           403M  5.5M  397M   2% /run
tmpfs          1006M     0 1006M   0% /dev/shm
tmpfs           5.0M     0  5.0M   0% /run/lock
tmpfs          1006M     0 1006M   0% /sys/fs/cgroup
```

Таким образом мы расширили корневой раздел работающей операционной системы Debian не прибегая к помощи LiveCD