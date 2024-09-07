---
title: "[РЕШЕНО] GPT структура разделов при автоматической установке  Debian 11 / Debian 10"
date: "2021-08-27"
categories:
  - Automation
tags: 
  - "debian"
  - "preseed"
coverImage: "no-release-repo.jpg"
image:
  path: /commons/no-release-repo.jpg
  alt: "GPT структура разделов"
---

> **GPT** — более новая и продвинутая структура разделов.
> При использовании MS-DOS partition table (MBR) на жёстком диске может быть сформировано 3 основных раздела (primary) и один дополнительный (extended). Загружаться можно только в режиме эмуляции BIOS. Ограничение на емкость диска 2 Tb.
> При использовании GUID partition table (GPT) на жёстком диске может быть сформировано 128 разделов, можно загружаться в режиме EFI. Ограничение на максимальный размер раздела — 9,4 ЗБ (зеттабайт).
{: .prompt-tip }

## Подготовка

- Ранее, в одной из статей, мы рассматривали [автоматическую установку Debian при помощи preseed]({% post_url 2019-12-17-avtomaticheskaya-ustanovka-debian-pri-pomoshhi-preseed %})

Для того, чтобы таблица разделов была в формате GPT, при создании конфигурационного preseed-файла надо добавить следующие строки:

В область `Disk partitioning`:

```
### Disk partitioning
d-i partman-auto/disk string /dev/sda
d-i partman-auto/method string lvm
d-i partman-lvm/device_remove_lvm boolean true
d-i partman-lvm/confirm boolean true
d-i partman-lvm/confirm_nooverwrite boolean true

d-i partman-basicfilesystems/choose_label string gpt
d-i partman-basicfilesystems/default_label string gpt
d-i partman-partitioning/choose_label string gpt
d-i partman-partitioning/default_label string gpt
d-i partman/choose_label string gpt
d-i partman/default_label string gpt
...
```

В область разбивки диска:

```
    1 1 1 free  \
        $primary{ }  \
        $bios_boot{}  \
        method{ biosgrub }  \
    .  \
```

Таким образом, кусок preseed-файла будет выглядеть следующим образом:

```
### Disk partitioning
d-i partman-auto/disk string /dev/sda
d-i partman-auto/method string lvm
d-i partman-lvm/device_remove_lvm boolean true
d-i partman-lvm/confirm boolean true
d-i partman-lvm/confirm_nooverwrite boolean true

d-i partman-basicfilesystems/choose_label string gpt
d-i partman-basicfilesystems/default_label string gpt
d-i partman-partitioning/choose_label string gpt
d-i partman-partitioning/default_label string gpt
d-i partman/choose_label string gpt
d-i partman/default_label string gpt

d-i partman-auto-lvm/new_vg_name string debian
d-i partman-auto-lvm/guided_size string max
#d-i partman-auto/choose_recipe select custom
d-i partman-auto/expert_recipe string  \
  custom ::  \
    1 1 1 free  \
        $primary{ }  \
        $bios_boot{}  \
        method{ biosgrub }  \
    .  \
    256 30 256 ext2  \
        $primary{ }  \
        $bootable{ }  \
        method{ format } format{ }  \
        use_filesystem{ } filesystem{ ext4 }  \
        mountpoint{ /boot }  \
    . \
    1024 1025 -1 xfs  \
        $lvmok{ }  lv_name{ root }  \
        method{ format } format{ }  \
        use_filesystem{ } filesystem{ xfs }  \
        mountpoint{ / }  \
    . \
#    1024 1025 -1 xfs  \
#        $lvmok{ }  lv_name{ var }  \
#        method{ format } format{ }  \
#        use_filesystem{ } filesystem{ xfs }  \
#        mountpoint{ /var }  \
#    . \
    1024 30 1024 linux-swap  \
        $lvmok{ } lv_name{ swap }  \
        method{ swap } format{ } \
    . 
```

## Проверка

Проверить тип таблицы разделов можно с помощью команды fdisk

```sh
$ sudo fdisk -l

Disk /dev/sda: 8 GiB, 8589934592 bytes, 16777216 sectors
Disk model: VBOX HARDDISK
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 2FD8E7D2-E382-43C0-9D78-8877EBCBBC2B

Device      Start      End  Sectors  Size Type
/dev/sda1    2048     4095     2048    1M BIOS boot
/dev/sda2    4096   503807   499712  244M Linux filesystem
/dev/sda3  503808 16775167 16271360  7.8G Linux LVM
```
