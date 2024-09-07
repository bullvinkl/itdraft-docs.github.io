---
title: "Монтируем FTP как папку в Сentos 6"
date: "2017-01-20"
categories: 
  - Storage-System
  - File-System
tags: 
  - "centos"
  - "ftp"
image:
  path: /commons/0_HICLyAdNSIyT0ODU-1.jpg
  alt: "Монтируем FTP как папку в Сentos 6"
---

> **FTP** (File Transfer Protocol) - это стандартный протокол для передачи файлов между клиентом и сервером через Интернет. Он позволяет пользователям загружать, скачивать и удалять файлы с удаленных серверов, а также управлять файлами на сервере.

Устанавливаем программу `curlftpfs` (Репозиторий EPEL)

```sh
$ sudo yum install curlftpfs
```

Создаем каталог, куда будем подключать FTP

```sh
$ sudo mkdir /mnt/ftpfolder
```

Монтируем

```sh
$ sudo curlftpfs -o allow_other ftpuser:ftppassword@itdraft.ru /mnt/ftpfolder
```

где
- `allow_other` - доступ к папке всех пользователей (по-умолчанию только root)
- `ftpuser` - логин от FTP
- `ftppassword` - пароль от FTP
- `itdraft.ru` - адрес FTP-сервера
- `/mnt/ftpfolder` - каталог, куда монтируется FTP
