---
title: "[РЕШЕНО] GPT структура разделов при авто установке (kickstart) Rocky Linux / Centos 8"
date: "2021-07-22"
categories: 
  - Linux
  - Kickstart
tags: 
  - "centos"
  - "kickstart"
  - "lvm"
  - "rocky-linux"
image:
  path: /commons/no-release-repo.jpg
  alt: ""
---

> **GPT** — более новая и продвинутая структура разделов.
> При использовании MS-DOS partition table (MBR) на жёстком диске может быть сформировано 3 основных раздела (primary) и один дополнительный (extended). Загружаться можно только в режиме эмуляции BIOS. Ограничение на емкость диска 2 Tb.
> При использовании GUID partition table (GPT) на жёстком диске может быть сформировано 128 разделов, можно загружаться в режиме EFI. Ограничение на максимальный размер раздела - 9,4 ЗБ (зеттабайт).
{: .prompt-tip }

## Настройка

Что бы таблица разделов была в формате GPT, для этого при создании конфигурационного kickstart-файла надо добавить/подправить следующие строки:

```sh
...
# Partition clearing information
zerombr
clearpart --all --initlabel --disklabel=gpt --drives=sda
...
# Disk partitioning information
part /boot --fstype="xfs" --size=200 --label="boot" --ondisk=sda
part biosboot --fstype="biosboot" --size=1 --ondisk=sda
#part /boot/efi --fstype="xfs" --size=200 --label="efi" --ondisk=sda
...
```

Т.е. самое основное: добавить параметр `--disklabel=gpt` в раздел `clearpart`, и добавить строку:

```
part biosboot --fstype="biosboot" --size=1 --ondisk=sda
```

без нее во время предустановки ОС появится ошибка в разделе разметки диска

## Проверка

Проверка определенного диска

```sh
$ sudo parted /dev/sda print
Model: ATA VBOX HARDDISK (scsi)
Disk /dev/sda: 9123MB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags:
```

Проверка всех дисков в системе через утилиту parted

```sh
$ sudo parted -l
Model: ATA VBOX HARDDISK (scsi)
Disk /dev/sda: 9123MB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags:
...
```

Проверка всех дисков в системе через утилиту fdisk

```sh
$ sudo fdisk -l
Disk /dev/sda: 8.5 GiB, 9122611200 bytes, 17817600 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 811BC132-5E8A-4EF1-9713-5E7549D301B7
...
```

Проверка всех дисков в системе через утилиту blkid

```sh
$ sudo blkid /dev/sda
/dev/sda: PTUUID="811bc132-5e8a-4ef1-9713-5e7549d301b7" PTTYPE="gpt"
```

## Тестирование

Я проверял добавление разделов через VirtualBox, следующим образом:

- Выключается виртуальная машина

- Увеличивается VDI-диск через командную строку:

```
"C:\Program Files\Oracle\VirtualBox\VBoxManage.exe" showhdinfo "C:\Users\user\VirtualBox VMs\testks\testks.vdi"
...
Capacity:       8192 MBytes
Size on disk:   2401 MBytes
...

"C:\Program Files\Oracle\VirtualBox\VBoxManage.exe" modifyhd "C:\Users\user\VirtualBox VMs\testks\testks.vdi" --resize 8500
```

- Запускается виртуальная машина и расширяется корневой LVM-раздел:

```sh
$ lsblk
$ sudo cfdisk /dev/sda
	New
	Type: Linux LVM (8e)
	Write
	Quit

$ lsblk
$ sudo pvcreate /dev/sda4 
$ sudo vgextend centos /dev/sda4
$ sudo lvextend  /dev/centos/root -l 100%VG
$ sudo xfs_growfs -d /dev/mapper/centos-root
```
