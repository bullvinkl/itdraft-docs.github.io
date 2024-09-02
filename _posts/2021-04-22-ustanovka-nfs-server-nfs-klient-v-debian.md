---
title: "Установка NFS-сервер / NFS-клиент в Debian"
date: "2021-04-22"
categories: 
  - Linux
  - NFS
tags: 
  - "nfs"
image:
  path: /commons/email-4044165_1280.jpg
  alt: "Установка NFS-сервер и NFS-клиент"
---

> **NFS (Network File System)** – сетевая файловая система, позволяющая пользователям обращаться к файлам и каталогам, расположенным на удалённых компьютерах. Более быстрый по сравнению с SAMBA и менее ресурсоемкий по сравнению с удаленными файловыми системами с шифрованием - sshfs, SFTP...

## Установка NFS-сервера

Обновляем список пакетов

```sh
$ sudo apt update
```

Устанавливаем NFS-сервер

```sh
$ sudo apt install nfs-kernel-server
```

Создаем каталог, который в дальнейшем будем расшаривать, и задаем права доступа

```sh
$ sudo mkdir /mnt/storage
$ sudo chmod 777 /mnt/storage/
```

Разрешаем сетевой доступ к каталогу для определенного клиента

```sh
$ sudo nano /etc/exports
/mnt/storage      192.168.10.8(rw,sync,no_root_squash,no_subtree_check)
```

Применяем настройки сетевого доступа

```sh
$ sudo exportfs -r
```

Проверяем

```sh
$ sudo systemctl status rpcbind nfs-server

$ sudo exportfs
/mnt/storage    192.168.10.8
```

## Установка NFS-клиента

Обновляем список пакетов

```sh
$ sudo apt update
```

Устанавливаем NFS-клиент

```sh
$ sudo apt install nfs-common
```

Запускаем службы

```sh
$ sudo systemctl start rpcbind
$ sudo systemctl enable rpcbind
```

Создаем точку монтирования

```sh
$ sudo mkdir /mnt/localstr
$ sudo chown user:user /mnt/localstr
```

Монтируем NFS-каталог

```sh
$ sudo mount -t nfs4 192.168.10.12:/mnt/storage /mnt/localstr
```

где:

- `192.168.10.12:/mnt/storage` - адрес NFS-сервера
- `/mnt/localstr` - локальная точка монтирования

Настраиваем автоматическое монтирование

```sh
$ sudo nano /etc/fstab
[...]
192.168.10.12:/mnt/storage    /mnt/localstp    nfs    defaults    0 0
```