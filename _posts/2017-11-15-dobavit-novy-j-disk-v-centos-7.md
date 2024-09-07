---
title: "Добавить новый диск в CentOS 7"
date: "2017-11-15"
categories: 
  - File-System
tags: 
  - "centos"
  - "fdisk"
  - "mkfs"
  - "mount"
image:
  path: /commons/1037058_b070_2.jpg
  alt: "Добавить новый диск в CentOS"
---

> **fdisk** - это утилита для управления разделами на жёстком диске в операционных системах семейства MS-DOS и Windows до версии XP. Она позволяет создавать, редактировать и удалять логические разделы на физическом диске.
{: .prompt-tip }

Добавляем диск в систему, проверяем, появился ли он:

```sh
$ sudo fdisk -l
```

Разбиваем диск на разделы

```sh
$ sudo fdisk /dev/sdb

Команда (m для справки): n
Partition type:
   p   primary (0 primary, 0 extended, 4 free)
   e   extended
Select (default p): p
Номер раздела (1-4, default 1): 1
Первый sector (2048-167772159, по умолчанию 2048): 
Используется значение по умолчанию 2048
Last sector, +sectors or +size{K,M,G} (2048-167772159, по умолчанию 167772159): 
Используется значение по умолчанию 167772159
Partition 1 of type Linux and of size 80 GiB is set

Команда (m для справки): w
Таблица разделов была изменена!

Вызывается ioctl() для перечитывания таблицы разделов.
Синхронизируются диски.
```

Где:  
- `n` - создать раздел  
- `p` - первичный раздел  
- `w` - сохранить изменения

Проверяем, должен появиться раздел `/dev/sdb1`

```sh
$ sudo fdisk -l /dev/sdb

Disk /dev/sdb: 85.9 GB, 85899345920 bytes, 167772160 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0xfbf1a157
```

Создаем на новом разделе файловую систему `ext4`

```sh
$ sudo mkfs -t ext4 /dev/sdb1

mke2fs 1.42.9 (28-Dec-2013)
Discarding device blocks: done                            
Filesystem label=
OS type: Linux
Block size=4096 (log=2)
Fragment size=4096 (log=2)
Stride=0 blocks, Stripe width=0 blocks
5242880 inodes, 20971264 blocks
1048563 blocks (5.00%) reserved for the super user
First data block=0
Maximum filesystem blocks=2168455168
640 block groups
32768 blocks per group, 32768 fragments per group
8192 inodes per group
Superblock backups stored on blocks: 
	32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208, 
	4096000, 7962624, 11239424, 20480000

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (32768 blocks): done
Writing superblocks and filesystem accounting information: done
```

Монтируем новый диск к каталогу `/opt/sdb` и смотрим резултат

```sh
$ sudo mkdir /opt/sdb
$ sudo mount -t ext4 /dev/sdb1 /opt/sdb
$ sudo df -h
Файловая система                         Размер Использовано  Дост Использовано% Cмонтировано в
/dev/mapper/centos_srv--tiledev--01-root    22G         1,4G   21G            7% /
devtmpfs                                   7,8G            0  7,8G            0% /dev
tmpfs                                      7,8G            0  7,8G            0% /dev/shm
tmpfs                                      7,8G          17M  7,8G            1% /run
tmpfs                                      7,8G            0  7,8G            0% /sys/fs/cgroup
/dev/sda1                                  497M         156M  342M           32% /boot
tmpfs                                      1,6G            0  1,6G            0% /run/user/0
/dev/sdb1                                   79G          57M   75G            1% /opt
```

Настраиваем автоматическое монтирование диска при старте системы, для этого редактируем `/etc/fstab` в самом конце добавляем:

```sh
$ sudo nano /etc/fstab
/dev/sdb1 /opt/sdb ext4 defaults 0 0
```
