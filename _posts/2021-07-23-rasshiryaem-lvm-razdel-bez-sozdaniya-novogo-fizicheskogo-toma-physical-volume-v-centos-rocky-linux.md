---
title: "Расширяем LVM-раздел без создания нового физического тома (physical volume) в Centos, Rocky Linux"
date: "2021-07-23"
categories: 
  - File-System
tags: 
  - centos
  - growpart
  - lvm"
  - pvresize
  - rocky-linux
  - xfs
  - ext4
image:
  path: /commons/digital-enablement.jpg
  alt: "Расширяем LVM-раздел без создания нового физического тома"
---

> При использовании MBR таблиц разделов есть ограничение: можно создать 3 основных раздела (primary)
> Утилита **growpart** — это инструмент для расширения разделов, который входит в пакет cloud utils.
{: .prompt-tip }

Смотрим структуру

```sh
$ lsblk
NAME            MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda               8:0    0    6G  0 disk
├─sda1            8:1    0    1M  0 part
├─sda2            8:2    0  200M  0 part /boot
sda3            8:3    0  5.8G  0 part
  ├─centos-root 253:0    0  5.3G  0 lvm  /
  └─centos-swap 253:1    0  512M  0 lvm  [SWAP]
```

В данном примере диск `sda` равен `6 Gb`

Выключаем машину, расширяем диск через виртуализацию, включаем машину

Устанавливаем утилиту `growpart` (Centos, Rocky Linux)

```sh
$ sudo yum -y install cloud-utils-growpart
```

Устанавливаем утилиту `growpart` (Debian 8)

```sh
$ sudo apt update
$ sudo apt -y install cloud-utils
```

Устанавливаем утилиту `growpart` (Debian 9 и выше)

```sh
$ sudo apt update
$ sudo apt -y install cloud-guest-utils
```

Расширяем раздел `3` на диске `/dev/sda`

```sh
$ sudo growpart /dev/sda 3
```

Смотрим, что получилось

```sh
$ lsblk
NAME            MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda               8:0    0  7.8G  0 disk
├─sda1            8:1    0    1M  0 part
├─sda2            8:2    0  200M  0 part /boot
sda3            8:3    0  7.6G  0 part
  ├─centos-root 253:0    0  5.3G  0 lvm  /
  └─centos-swap 253:1    0  512M  0 lvm  [SWAP]
```

Расширяем физический том (physical volume)

```sh
$ sudo pvresize /dev/sda3
  Physical volume "/dev/sda3" changed
  1 physical volume(s) resized or updated / 0 physical volume(s) not resized

```

Проверяем размер физического том (physical volume)

```sh
$ sudo pvs
  PV         VG     Fmt  Attr PSize PFree
  /dev/sda3  centos lvm2 a-- 7.61g 1.81g
```

Проверяем размер группы томов (volume group)

```sh
$ sudo vgs
  VG     #PV #LV #SN Attr   VSize VFree
  centos   1   2   0 wz--n- 7.61g 1.81g
```

Проверяем размер корневого размера, заодно смотрим путь и тип файловой системы (в данном примере xfs)

```sh
$ df -hT | grep mapper
Filesystem               Type      Size  Used Avail Use% Mounted on
/dev/mapper/centos-root  xfs       7.2G  1.7G  5.5G  23% /

```

Расширяем логический том (logical volume)

```sh
$ sudo lvextend -r -l +100%FREE /dev/mapper/centos-root
```

Расширяем файловую систему `XFS`

```sh
$ sudo xfs_growfs /
```

Либо, расширяем файловую систему `EXT4`

```sh
$ sudo resize2fs /dev/mapper/centos-root
```
