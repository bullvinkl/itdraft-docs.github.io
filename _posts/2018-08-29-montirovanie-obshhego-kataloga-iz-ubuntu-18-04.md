---
title: "Монтирование общего каталога из Ubuntu 18.04"
date: "2018-08-29"
categories: 
  - Linux
tags: 
  - "cifs-utils"
  - "mount"
  - "samba"
  - "ubuntu"
image:
  path: /commons/1039062_e553_4.jpg
  alt: "Монтирование общего каталога"
---

> **CIFS-utils** (Common Internet File System) - утилита для монтирования шары (сетевого ресурса) в Linux, позволяющая работать с файловой системой Windows (SMB) на уровне локальной файловой системы. Это означает, что вы можете монтировать шары Windows на ваш компьютер Linux, как если бы они были локальными дисками.
{: .prompt-tip }

Устанавливаем утилиту `cifs-utils`

```sh
$ sudo apt-get install cifs-utils
```

Создаем каталог, куда будем монтировать общий каталог

```sh
$ sudo mkdir /mnt/shred
```

Монтируем

```sh
$ sudo mount -t cifs //192.168.1.33/SharedFolder /mnt/shred -o username="user",password="pass",domain="DOMAIN",vers=1.0
```

Для постоянного подключения к общему каталогу (при старте системы) добавляем запись в файл `/etc/fstab`

```sh
$ sudo nano /etc/fstab
...
//192.168.1.33/SharedFolder/   /mnt/linux_smb   cifs   username=user,password=pass,domain=DOMAIN,vers=1.0   0    0
```
