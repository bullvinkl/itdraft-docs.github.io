---
title: "Монтируем диск в NTFS"
date: "2015-07-30"
categories: 
  - File-System
tags: 
  - "centos"
  - "linux"
  - "mount"
  - "ntfs"
  - "unmount"
image:
  path: /commons/template-1599665_1280.png
  alt: "Монтируем диск в NTFS"
---

> **NTFS** (New Technology File System) - это файловая система, разработанная компанией Microsoft для операционных систем Windows NT и его последующих версий. Она была создана для обеспечения безопасности, эффективности и масштабируемости файловой системы.
{: .prompt-tip }

Устанавливаем пакеты для `CentOS 5.x`

```sh
$ sudo yum install ntfs-3g
```

Устанавливаем пакеты для `CentOS 6.x`

```sh
$ sudo yum install fuse fuse-ntfs-3g dkms dkms-fuse
```

Смотрим информацию о дисках

```sh
$ sudo fdisk -l
```

Создаем каталог, куда монтировать

```sh
$ sudo mkdir /mnt/sdb1
```

Монтируем

```sh
$ sudo mount -t ntfs-3g /dev/sdb1 /mnt/sdb1
```

Размонтируем

```sh
$ sudo umount /mnt/sdb1
```
