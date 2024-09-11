---
layout: post
title: "Добавляем LVM диск в Ubuntu"
date: "2024-09-11"
categories:
  - File-System
tags:
  - ubuntu
  - lvm
  - uuid
  - ext4
image:
  path: /commons/warning01.png
  alt: "Добавляем LVM диск в Ubuntu"
---

> Logical Volume Manager (LVM) - это очень мощная система управления томами с данными для Linux. Она позволяет создавать поверх физических разделов (или даже неразбитых винчестеров) логические тома, которые в самой системе будут видны как обычные блочные устройства с данными (т. е. как обычные разделы).
{: .prompt-tip }

Подключаем диск, размечаем его
```sh
$ sudo cfdisk /dev/sdb
```

- `gpt`
- `new > partition size: отставляем значение`
- `type > Linux LVM`
- `write > yes`
- `quit`

Инициализируем диск, создаем группа томов `vg02`, создаем логический раздел `data` в группе томов `vg02`

```sh
$ sudo pvcreate /dev/sdb1
$ sudo vgcreate vg02 /dev/sdb1
$ sudo lvcreate -n data -l 100%FREE vg02
```

Создаем файловую систему `ext4`
```sh
$ sudo mkfs.ext4 /dev/vg02/data
```

Создаем точку монтирования
```sh
$ sudo mkdir /opt/data
```

Ищем наш UUID раздела
```sh
$ ls -l /dev/disk/by-id/
...
lrwxrwxrwx 1 root root 10 Sep 11 16:20 dm-uuid-LVM-BDOtt3j0bgkJuI6V25f7dD6wk8dxmD6lO5xFbCreu3Q9dVrXUFWKHjHYoNaIzWzV -> ../../dm-1
lrwxrwxrwx 1 root root 10 Sep  9 16:05 dm-uuid-LVM-qdG0Fijey8tCXUKxArR2L4EWdJGaT0CJN87SAdGFsd5U5Eh7F1n3b3TXMpZZnnIT -> ../../dm-0
...
```

- `dm-uuid-LVM-BDOtt3j0bgkJuI6V25f7dD6wk8dxmD6lO5xFbCreu3Q9dVrXUFWKHjHYoNaIzWzV -> ../../dm-1`  - наш диск

Либо второй способ
```sh
$ sudo vgscan -vvv |& less
...
  Found dev 253:0 /dev/disk/by-dname/ubuntu--vg-ubuntu--lv - new alias.
  Found dev 253:0 /dev/disk/by-id/dm-name-ubuntu--vg-ubuntu--lv - new alias.
  Found dev 253:0 /dev/disk/by-id/dm-uuid-LVM-qdG0Fijey8tCXUKxArR2L4EWdJGaT0CJN87SAdGFsd5U5Eh7F1n3b3TXMpZZnnIT - new alias.
  Found dev 253:0 /dev/disk/by-uuid/5d2703b8-e8ce-4bea-9fc6-eeca9aefe9b6 - new alias.
  Found dev 253:0 /dev/mapper/ubuntu--vg-ubuntu--lv - new alias.
  Found dev 253:0 /dev/ubuntu-vg/ubuntu-lv - new alias.
  Found dev 253:1 /dev/dm-1 - new.
  Found dev 253:1 /dev/disk/by-id/dm-name-vg02-data - new alias.
  Found dev 253:1 /dev/disk/by-id/dm-uuid-LVM-BDOtt3j0bgkJuI6V25f7dD6wk8dxmD6lO5xFbCreu3Q9dVrXUFWKHjHYoNaIzWzV - new alias.
  Found dev 253:1 /dev/disk/by-uuid/b70e8589-780e-422c-b5a2-c06aa117a79d - new alias.
  Found dev 253:1 /dev/mapper/vg02-data - new alias.
  Found dev 253:1 /dev/vg02/data - new alias.
...
```

`Found dev 253:1 /dev/disk/by-id/dm-uuid-LVM-BDOtt3j0bgkJuI6V25f7dD6wk8dxmD6lO5xFbCreu3Q9dVrXUFWKHjHYoNaIzWzV - new alias.`  - виден весь путь, который пропишем в `/etc/fstab`

Редактируем файл `/etc/fstab` для авто монтирования раздела при загрузке ОС
```sh
$ sudo nano /etc/fstab
...
/dev/disk/by-id/dm-uuid-LVM-BDOtt3j0bgkJuI6V25f7dD6wk8dxmD6lO5xFbCreu3Q9dVrXUFWKHjHYoNaIzWzV /opt/data ext4 defaults 0 1
```

Монтируем
```sh
$ sudo systemctl daemon-reload
$ sudo mount -a
```

Проверяем
```sh
$ sudo mount | grep mapper
$ lsblk
```
