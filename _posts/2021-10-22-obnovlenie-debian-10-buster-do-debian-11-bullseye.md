---
title: "Обновление Debian 10 (buster) до Debian 11 (bullseye)"
date: "2021-10-22"
categories: 
  - Manuals
tags: 
  - "debian"
  - "upgrade"
image:
  path: /commons/virtualisationimage.png
  alt: "Обновление Debian 10 (buster) до Debian 11 (bullseye)"
---

> **Debian 11** кодовое имя "bullseye". Дата релиза 15.08.2021.
> Основные изменения:
> - Ядро Linux обновлено до версии 5.10 (в Debian 10 поставлялось ядро 4.19);
> - Обновлены серверные приложения, в том числе Apache httpd 2.4.48, BIND 9.16, Dovecot 2.3.13, Exim 4.94, Postfix 3.5, MariaDB 10.5, nginx 1.18, PostgreSQL 13, Samba 4.13, OpenSSH 8.4;
> - Обновлены серверные приложения, в том числе Apache httpd 2.4.48, BIND 9.16, Dovecot 2.3.13, Exim 4.94, Postfix 3.5, MariaDB 10.5, nginx 1.18, PostgreSQL 13, Samba 4.13, OpenSSH 8.4;
> - Изменён формат строк в файле /etc/apt/sources.list, связанных с устранением проблем с безопасностью. Строки {dist}-updates переименованы в {dist}-security. В sources.list разрешено отделение блоков "[]" несколькими пробелами;
{: .prompt-tip }

Для начала обновим текущую систему

```sh
$ sudo apt update 
$ sudo apt upgrade -y
$ sudo apt autoremove -y
$ sudo apt dist-upgrade -y
```

Меняем репозиторий buster на bullseye

```sh
$ sudo nano /etc/apt/sources.list
### Debian 11 (bullseye)
deb http://deb.debian.org/debian bullseye main contrib non-free
deb-src http://deb.debian.org/debian bullseye main contrib non-free

deb http://deb.debian.org/debian-security/ bullseye-security main contrib non-free
deb-src http://deb.debian.org/debian-security/ bullseye-security main contrib non-free

deb http://deb.debian.org/debian bullseye-updates main contrib non-free
deb-src http://deb.debian.org/debian bullseye-updates main contrib non-free
```

Если в системе подключены другие репозитории, не забываем поменять и их

Обновляем список доступных пакетов для нового выпуска

```sh
$ sudo apt update
```

Обновление системы будем делать в 2 этапа.

Сначала минимальное обновление

```sh
$ sudo apt upgrade -y
```

После завершения минимального обновления, запускаем полное

```sh
$ sudo apt dist-upgrade -y
```

Перезгружаемся

```sh
$ sudo reboot
```

Чистим мусор

```sh
$ sudo apt autoremove -y
```

Проверяем версию ОС

```sh
$ cat /etc/*release
```
