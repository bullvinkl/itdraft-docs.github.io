---
title: "Как увеличить LVM раздел в CentOS 7"
date: "2017-05-29"
categories: 
  - File-System
tags: 
  - "centos"
  - "lvm"
image:
  path: /commons/910838_84d6.jpg
  alt: "Как увеличить LVM раздел"
---

> **Логический том** (LVM) - это система управления дисковым пространством в операционных системах Linux и OS/2, позволяющая абстрагироваться от физических устройств и создавать логические тома, которые могут быть изменены в размере, перемещены и объединены в онлайн-режиме.
{: .prompt-tip }

В виртуальной машине на ESXi есть CentOS 7, надо увеличить раздел centos-root

Проверяем, какие разделы есть в операционный системе

```sh
$ sudo fdisk -l
Disk /dev/sda: 69.1 GB, 118111600640 bytes, 230686720 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x000811c8

Устр-во Загр Начало Конец Блоки Id Система
/dev/sda1 * 2048 1026047 512000 83 Linux
/dev/sda2 1026048 125829119 62401536 8e Linux LVM

Disk /dev/mapper/centos-root: 38.6 GB, 38562430976 bytes, 75317248 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

Disk /dev/mapper/centos-swap: 6442 MB, 6442450944 bytes, 12582912 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

Disk /dev/mapper/centos-home: 18.8 GB, 18828230656 bytes, 36773888 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
```

Выключаем операционную систему, добавляем через админку ESXi дополнительное пространство, загружаемся.

Смотрим, что получилось:

```sh
$ sudo fdisk -l
Disk /dev/sda: 118.1 GB, 118111600640 bytes, 230686720 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x000811c8

Устр-во Загр Начало Конец Блоки Id Система
/dev/sda1 * 2048 1026047 512000 83 Linux
/dev/sda2 1026048 125829119 62401536 8e Linux LVM

Disk /dev/mapper/centos-root: 38.6 GB, 38562430976 bytes, 75317248 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

Disk /dev/mapper/centos-swap: 6442 MB, 6442450944 bytes, 12582912 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

Disk /dev/mapper/centos-home: 18.8 GB, 18828230656 bytes, 36773888 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
```

Размер диска `/deb/sda` увеличился с `69.1` GB до `118.1` GB

Проверяем, сколько места на диске занято

```sh
$ sudo df -h
Файловая система Размер Использовано Дост Использовано% Cмонтировано в
/dev/mapper/centos-root 36G 34G 2,5G 94% /
devtmpfs 7,8G 0 7,8G 0% /dev
tmpfs 7,8G 88K 7,8G 1% /dev/shm
tmpfs 7,8G 8,9M 7,8G 1% /run
tmpfs 7,8G 0 7,8G 0% /sys/fs/cgroup
/dev/mapper/centos-home 18G 37M 18G 1% /home
/dev/sda1 497M 159M 339M 32% /boot
tmpfs 1,6G 28K 1,6G 1% /run/user/0
/dev/sr0 7,3G 7,3G 0 100% /run/media/root/CentOS 7 x86_64
```

Видно, что в системе место не увеличилось

У нас появилась не размеченная область, создадим новый раздел с типом Linux LVM

```sh
$ sudo fdisk /dev/sda
Команда (m для справки): n (добавление нового раздела)
Partition type:
p primary (2 primary, 0 extended, 2 free)
e extended
Select (default p): p (обозначаем его как primary)
Номер раздела (3,4, default 3): 3
Первый sector (125829120-230686719, по умолчанию 125829120):
Используется значение по умолчанию 125829120
Last sector, +sectors or +size{K,M,G} (125829120-230686719, по умолчанию 230686719):
Используется значение по умолчанию 230686719
Partition 3 of type Linux and of size 50 GiB is set

Команда (m для справки): t (укажем тип раздела)
Номер раздела (1-3, default 3): 3
Hex code (type L to list all codes): 8e (код типа раздела)
Changed type of partition 'Linux' to 'Linux LVM'

Команда (m для справки): p (вывод таблицы разделов)

Disk /dev/sda: 118.1 GB, 118111600640 bytes, 230686720 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x000811c8

Устр-во Загр Начало Конец Блоки Id Система
/dev/sda1 * 2048 1026047 512000 83 Linux
/dev/sda2 1026048 125829119 62401536 8e Linux LVM
/dev/sda3 125829120 230686719 52428800 8e Linux LVM

Команда (m для справки): w (запись таблицы разделов на диск и выход)
Таблица разделов была изменена!
```

Мы создали раздел `/dev/sda3`, перезагружаемся (не обязательно) и проверяем

```sh
$ sudo reboot
$ sudo fdisk -l

Disk /dev/sda: 118.1 GB, 118111600640 bytes, 230686720 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x000811c8

Устр-во Загр Начало Конец Блоки Id Система
/dev/sda1 * 2048 1026047 512000 83 Linux
/dev/sda2 1026048 125829119 62401536 8e Linux LVM
/dev/sda3 125829120 230686719 52428800 8e Linux LVM

Disk /dev/mapper/centos-root: 38.6 GB, 38562430976 bytes, 75317248 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

Disk /dev/mapper/centos-swap: 6442 MB, 6442450944 bytes, 12582912 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

Disk /dev/mapper/centos-home: 18.8 GB, 18828230656 bytes, 36773888 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
```

Создадим физический том `sda3` через команду `pvcreate`

```sh
$ sudo pvcreate /dev/sda3
Physical volume "/dev/sda3" successfully created
```

Ищем наш `VG Name`:

```sh
$ sudo vgdisplay
--- Volume group ---
VG Name centos
System ID
Format lvm2
Metadata Areas 1
Metadata Sequence No 4
VG Access read/write
VG Status resizable
MAX LV 0
Cur LV 3
Open LV 3
Max PV 0
Cur PV 1
Act PV 1
VG Size 59,51 GiB
PE Size 4,00 MiB
Total PE 15234
Alloc PE / Size 15219 / 59,45 GiB
Free PE / Size 15 / 60,00 MiB
VG UUID GO9ZOb-mjql-g4uc-x7DL-8L8A-Fu8Z-z9D1nA
```

Расширяем группу томов

```sh
$ sudo vgextend centos /dev/sda3
Volume group "centos" successfully extended
```

Проверяем

```sh
$ sudo pvscan
PV /dev/sda2 VG centos lvm2 [59,51 GiB / 60,00 MiB free]
PV /dev/sda3 VG centos lvm2 [50,00 GiB / 50,00 GiB free]
Total: 2 [109,50 GiB] / in use: 2 [109,50 GiB] / in no VG: 0 [0 ]
```

Ищем наш `LV Path`:

```sh
$ sudo lvdisplay
--- Logical volume ---
LV Path /dev/centos/swap
LV Name swap
VG Name centos
LV UUID kEa2eH-JnhM-NZRb-ifeS-kV2Q-PpuT-cn3DWs
LV Write Access read/write
LV Creation host, time localhost, 2017-04-11 16:55:10 +0300
LV Status available
# open 2
LV Size 6,00 GiB
Current LE 1536
Segments 1
Allocation inherit
Read ahead sectors auto
- currently set to 8192
Block device 253:1

--- Logical volume ---
LV Path /dev/centos/home
LV Name home
VG Name centos
LV UUID oRAZAF-dBB3-JRaq-Y2Kb-Q7lo-2Rah-9lB28G
LV Write Access read/write
LV Creation host, time localhost, 2017-04-11 16:55:10 +0300
LV Status available
# open 1
LV Size 17,54 GiB
Current LE 4489
Segments 1
Allocation inherit
Read ahead sectors auto
- currently set to 8192
Block device 253:2

--- Logical volume ---
LV Path /dev/centos/root
LV Name root
VG Name centos
LV UUID NTrhMk-mZV0-cpjK-13lR-L2Ji-DzeG-f6e5vY
LV Write Access read/write
LV Creation host, time localhost, 2017-04-11 16:55:10 +0300
LV Status available
# open 1
LV Size 35,91 GiB
Current LE 9194
Segments 1
Allocation inherit
Read ahead sectors auto
- currently set to 8192
Block device 253:0
```

Расширяем логический том и активируем его

```sh
$ sudo lvextend -l+100%FREE /dev/centos/root
Size of logical volume centos/root changed from 35,91 GiB (9194 extents) to 85,97 GiB (22008 extents).
Logical volume root successfully resized.
$ sudo vgscan
Reading all physical volumes. This may take a while...
Found volume group "centos" using metadata type lvm2
[root@srv-routing-01 ~]# vgchange -ay
3 logical volume(s) in volume group "centos" now active
```

Расширяем файловую систему `xfs`

```sh
$ sudo xfs_growfs /dev/mapper/centos-root
meta-data=/dev/mapper/centos-root isize=256 agcount=4, agsize=2353664 blks
= sectsz=512 attr=2, projid32bit=1
= crc=0 finobt=0
data = bsize=4096 blocks=9414656, imaxpct=25
= sunit=0 swidth=0 blks
naming =version 2 bsize=4096 ascii-ci=0 ftype=0
log =internal bsize=4096 blocks=4597, version=2
= sectsz=512 sunit=0 blks, lazy-count=1
realtime =none extsz=4096 blocks=0, rtextents=0
data blocks changed from 9414656 to 22536192
```

Расширяем файловую систему `ext4`

```sh
$ sudo resize2fs /dev/mapper/centos-root
```

Проверяем

```sh
$ sudo df -h
Файловая система Размер Использовано Дост Использовано% Cмонтировано в
/dev/mapper/centos-root 86G 34G 53G 39% /
devtmpfs 7,8G 0 7,8G 0% /dev
tmpfs 7,8G 84K 7,8G 1% /dev/shm
tmpfs 7,8G 8,9M 7,8G 1% /run
tmpfs 7,8G 0 7,8G 0% /sys/fs/cgroup
/dev/mapper/centos-home 18G 37M 18G 1% /home
/dev/sda1 497M 159M 339M 32% /boot
tmpfs 1,6G 12K 1,6G 1% /run/user/0
/dev/sr0 7,3G 7,3G 0 100% /run/media/root/CentOS 7 x86_64
```

## UPD 2017.11.02

в команде: `xfs_growfs /dev/mapper/centos-root` значение `/dev/mapper/centos-root` мы берем выполнив команду `df -h`
Какой раздел расширяем, то значение и пишем
