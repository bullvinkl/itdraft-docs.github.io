---
title: "Увеличить диск c GPT-разметкой при помощи cfdisk в Linux"
date: "2021-05-12"
categories: 
  - File-System
tags: 
  - "cfdisk"
  - "ext4"
  - "fdisk"
  - "gpt"
image:
  path: /commons/mockup-2443050_1280.jpg"
  alt: "Увеличить диск c GPT-разметкой при помощи cfdisk"
---

> **cfdisk** — системная утилита для управления разделами жёсткого диска в операционной системе Linux. Схожа с fdisk, но имеет дружелюбный пользовательский интерфейс
{: .prompt-tip }

Задача: требуется увеличить размер GPT-диска `/dev/sdb1` (тип файловой системы `ext4`)

- Выключаем виртуалку
- Расширяем нужный диск
- Включаем виртуалку

Останавливаем все службы, которые хранят данные в примонтированном диске

```sh
$ sudo systemctl stop zabbix-server
$ sudo systemctl stop postgresql@9.6-main.service
```

Размонтируем диск

```sh
$ sudo umount /mnt/data
```

Проверяем

```sh
$ lsblk
```

Запускаем утилиту `cfdisk`

```sh
$ sudo cfdisk /dev/sdb
```

![](/assets/img/posts/2021/05/12/cfdisk01.png){: w="300" }
_cfdisk /dev/sdb_

Появилось свободное пространство

![](/assets/img/posts/2021/05/12/cfdisk03.png){: w="300" }
_Resize_

Выбираем опцию `Resize`, указываем размер (по умолчанию будет размер всего диска), записываем изменения (опция `write`) и выходим

Плюсы утилиты `cfdisk`:

- она корректно работает с gpt-разметкой,
- установлена по умолчанию в дистрибутивах Debian 8+, Centos 7+,
- не надо удалять раздел, как это делается в `fdisk`

Монтируем разделы, которые до этого размантировали

```sh
$ sudo mount -a
```

Проверяем, что диск расширился

```sh
$ lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
fd0      2:0    1    4K  0 disk 
sda      8:0    0    8G  0 disk 
├─sda1   8:1    0  243M  0 part /boot
├─sda2   8:2    0  1.9G  0 part [SWAP]
└─sda3   8:3    0  5.9G  0 part /
sdb      8:16   0   36G  0 disk 
└─sdb1   8:17   0   36G  0 part /mnt/data
sr0     11:0    1 1024M  0 rom
```

Как видно, `sdb1` стал 36 Gb

Проверяем количество занятого места на диске

```sh
$ df -h
Filesystem      Size  Used Avail Use% Mounted on
udev            2.9G     0  2.9G   0% /dev
tmpfs           597M  8.1M  589M   2% /run
/dev/sda3       5.7G  2.7G  2.8G  49% /
tmpfs           3.0G   16K  3.0G   1% /dev/shm
tmpfs           5.0M     0  5.0M   0% /run/lock
tmpfs           3.0G     0  3.0G   0% /sys/fs/cgroup
/dev/sda1       232M   71M  145M  33% /boot
tmpfs           597M     0  597M   0% /run/user/1002
/dev/sdb1        16G   12G  3.0G  81% /mnt/data
```

Тут отображается размер файловой системы 16 Gb

Расширяем (для `ext4`)

```sh
$ sudo resize2fs /dev/sdb1
resize2fs 1.44.5 (15-Dec-2018)
Filesystem at /dev/sdb1 is mounted on /mnt/data; on-line resizing required
old_desc_blocks = 1, new_desc_blocks = 3
The filesystem on /dev/sdb1 is now 9436919 (4k) blocks long.
```

Проверяем

```sh
$ df -h
Filesystem      Size  Used Avail Use% Mounted on
udev            2.9G     0  2.9G   0% /dev
tmpfs           597M  8.1M  589M   2% /run
/dev/sda3       5.7G  2.7G  2.8G  49% /
tmpfs           3.0G   16K  3.0G   1% /dev/shm
tmpfs           5.0M     0  5.0M   0% /run/lock
tmpfs           3.0G     0  3.0G   0% /sys/fs/cgroup
/dev/sda1       232M   71M  145M  33% /boot
tmpfs           597M     0  597M   0% /run/user/1002
/dev/sdb1        36G   12G   22G  36% /mnt/data
```

Как видно, раздел `/dev/sdb1` стал 36 Gb

Запускаем сервисы, которые мы останавливали ранее

```sh
$ sudo systemctl start postgresql@9.6-main.service
$ sudo systemctl start zabbix-server
```
