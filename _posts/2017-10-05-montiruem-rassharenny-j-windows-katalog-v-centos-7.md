---
title: "Монтируем расшаренный Windows-каталог в CentOS 7"
date: "2017-10-05"
categories: 
  - Linux
tags: 
  - "centos"
  - "mount"
  - "samba"
  - "smb"
image:
  path: /commons/986254_f32a_2.jpg
  alt: "Монтируем расшаренный Windows-каталог в CentOS"
---

> **CIFS-utils** (Common Internet File System utilities) - это набор утилит для монтирования и управления файловыми ресурсами, доступными через протокол CIFS (SMB). Протокол CIFS позволяет подключаться к файловым ресурсам, расположенным на других компьютерах, включая Windows-серверы.
{: .prompt-tip }

Устанавливаем утилиту `cifs-utils`

```sh
$ sudo yum install cifs-utils
```

Добавляем пользователей

```sh
$ sudo useradd -u 5000 UserPackages
$ sudo useradd -u 5001 UserTraffic
$ sudo groupadd -g 6000 share_library
$ sudo usermod -G share_library -a UserPackages
$ sudo usermod -G share_library -a UserTraffic
```

Создаем каталоги, в которые будем монтировать расшаренные windows-ресурсы

```sh
$ sudo mkdir /mnt/Packages
$ sudo mkdir /mnt/Traffic
```

Создаем файл с настройками доступа к расшаренным windows-ресурсам и задаем права на этот файл

```sh
$ sudo nano /root/smb_user
username=user
domain=DOMAIN
password=password
$ sudo chmod 0600 /root/smb_user
```

Монтируем

```sh
$ sudo mount.cifs \\192.168.0.15\Packages /mnt/Packages -o credentials=/root/smb_user,uid=5000,gid=6000
$ sudo mount.cifs \\192.168.0.15\Traffic /mnt/Traffic -o credentials=/root/smb_user,uid=5001,gid=6000
```

Команды для размонтирования:

```sh
$ sudo umount /mnt/Packages
$ sudo umount /mnt/Traffic
```

## Автоматическое монтирование каталогов при последующей загрузке операционной системы

Открываем файл `/etc/fstab` в текстовом редакторе и добавляем в конце файла строки:

```sh
$ sudo nano /etc/fstab
\\192.168.0.15\Packages /mnt/Packages    cifs    credentials=/root/smb_user,uid=5000,gid=6000 0 0
\\192.168.0.15\Traffic /mnt/Traffic   cifs    credentials=/root/smb_user,uid=5001,gid=6000 0 0
```
