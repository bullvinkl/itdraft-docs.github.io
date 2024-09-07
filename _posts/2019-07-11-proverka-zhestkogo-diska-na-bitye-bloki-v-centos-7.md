---
title: "Проверка жесткого диска на битые блоки в Centos 7"
date: "2019-07-11"
categories: 
  - Manuals
tags: 
  - "badblocks"
  - "centos"
image:
  path: /commons/0_HICLyAdNSIyT0ODU-1.jpg
  alt: "Проверка жесткого диска на битые блоки"
---

> Повреждённый сектор (англ. bad sector, bad block, повреждённый блок, в любительской литературе — бэд-сектор) — сбойный (не читающийся) или ненадежный сектор жёсткого диска или флэш-накопителя; кластер, содержащий сбойные сектора, или кластер помеченный таковым в структурах файловой системы (операционной системой, дисковой утилитой или же вирусом, для собственного использования)
{: .prompt-tip }

Для проверки диска на битые блоки в операционной системе предустановлена утилита `badblocks`

Смотрим разметку диска:

```sh
$ sudo fdisk -l

Disk /dev/sda: 240.1 GB, 240057409536 bytes, 468862128 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x00075d8d

   Device Boot      Start         End      Blocks   Id  System
/dev/sda1   *        2048     1173503      585728   83  Linux
/dev/sda2         1173504     9969663     4398080   8e  Linux LVM
/dev/sda3         9969664   468860927   229445632   fd  Linux raid autodetect

Disk /dev/sdb: 240.1 GB, 240057409536 bytes, 468862128 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x000a3965

   Device Boot      Start         End      Blocks   Id  System
/dev/sdb1            2048     8798207     4398080   8e  Linux LVM
/dev/sdb2         8798208   467689471   229445632   fd  Linux raid autodetect

Disk /dev/md127: 234.8 GB, 234818109440 bytes, 458629120 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

Disk /dev/mapper/centos-swap: 9000 MB, 9000976384 bytes, 17580032 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
```

Проверяем диски:

```sh
$ sudo badblocks -v /dev/sda3 -s
Checking blocks 0 to 229445631
Checking for bad blocks (read-only test): done
Pass completed, 0 bad blocks found. (0/0/0 errors)

$ sudo badblocks -v /dev/sda2 -s
Checking blocks 0 to 4398079
Checking for bad blocks (read-only test): done
Pass completed, 0 bad blocks found. (0/0/0 errors)

$ sudo badblocks -v /dev/sda1 -s
Checking blocks 0 to 585727
Checking for bad blocks (read-only test): done
Pass completed, 0 bad blocks found. (0/0/0 errors)
```

Параметр `-s` нужен для отображения прогресса проверки

Как видно из лога, битые блоки на диске не обнаружены
