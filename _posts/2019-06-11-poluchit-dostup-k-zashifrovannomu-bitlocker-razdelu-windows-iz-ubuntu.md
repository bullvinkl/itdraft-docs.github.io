---
title: "Получить доступ к зашифрованному BitLocker разделу Windows из Ubuntu"
date: "2019-06-11"
categories: 
  - Windows
  - Linux
tags: 
  - "bitlocker"
  - "dislocker"
  - "encrypt"
  - "mount"
  - "ubuntu"
image:
  path: /commons/629418_a5b0.jpg
  alt: "Получить доступ к зашифрованному BitLocker разделу Windows из Ubuntu"
---

> **BitLocker** — это функция защиты данных, которая шифрует диски на компьютере, чтобы предотвратить хищение и разглашение данных. Проприетарная технология шифрования дисков, являющаяся частью операционных систем Microsoft Windows

Для того, чтобы получить доступ к зашифрованному Bitlocker разделу Windows, например к зашифрованной USB-flash карте, для начала надо установить утилиту `DisLocker`

![](/assets/img/posts/2019/06/11/wp_bitlocker_1.png){: w="300" }

```sh
$ sudo apt install dislocker
```

Прежде чем начать дешифровку, создадим две директории

- В первой директории будет находиться виртуальный "файл" - примонтированный NTFS-раздел
- Вторая директория - туда мы будем монтировать наш дешифрованный радел, т.е. первую директорию

```sh
$ sudo mkdir -p /media/bitlocker
$ sudo mkdir -p /media/mount
```

Подключим нашу зашифрованную флэшку к ПК и через утилиту `fdisk` определим имя устройства

```sh
$ sudo fdisk -l

Диск /dev/sdb: 14,5 GiB, 15527313408 байт, 30326784 секторов
Disk model: Silicon-Power16G
Единицы: секторов по 1 * 512 = 512 байт
Размер сектора (логический/физический): 512 байт / 512 байт
Размер I/O (минимальный/оптимальный): 512 байт / 512 байт
Тип метки диска: dos
Идентификатор диска: 0x7f2905be

Устр-во    Загрузочный начало    Конец  Секторы Размер Идентификатор Тип
/dev/sdb1                8064 30326783 30318720  14,5G             7 HPFS/NTFS/exFAT
```

Дешифруем наш раздел `/dev/sdb1`

```sh
$ sudo dislocker -V /dev/sdb1 -u%password% -- /media/bitlocker
```

где
- `/dev/sdb1` - наше зашифрованное устройство
- `%password%` - пароль шифрования
- `/media/bitlocker` - каталог монтирования


Другие поддерживаемые команды для утилиты `dislocker` можно узнать, выполнив:

```sh
$ dislocker -h
dislocker by Romain Coltel, v0.7.1 (compiled for Linux/x86_64)

Usage: dislocker [-hqrsv] [-l LOG_FILE] [-O OFFSET] [-V VOLUME DECRYPTMETHOD -F[N]] [-- ARGS...]
    with DECRYPTMETHOD = -p[RECOVERY_PASSWORD]|-f BEK_FILE|-u[USER_PASSWORD]|-k FVEK_FILE|-c

Options:
    -c, --clearkey        decrypt volume using a clear key (default)
    -f, --bekfile BEKFILE
                          decrypt volume using the bek file (on USB key)
    -F, --force-block=[N] force use of metadata block number N (1, 2 or 3)
    -h, --help            print this help and exit
    -k, --fvek FVEK_FILE  decrypt volume using the FVEK directly
    -l, --logfile LOG_FILE
                          put messages into this file (stdout by default)
    -O, --offset OFFSET   BitLocker partition offset, in bytes (default is 0)
    -p, --recovery-password=[RECOVERY_PASSWORD]
                          decrypt volume using the recovery password method
    -q, --quiet           do NOT display anything
    -r, --readonly        do not allow one to write on the BitLocker volume
    -s, --stateok         do not check the volume's state, assume it's ok to mount it
    -u, --user-password=[USER_PASSWORD]
                          decrypt volume using the user password method
    -v, --verbosity       increase verbosity (CRITICAL errors are displayed by default)
    -V, --volume VOLUME   volume to get metadata and keys from

    -- end of program options, beginning of FUSE's ones

  ARGS are any arguments you want to pass to FUSE. You need to pass at least
the mount-point.
```

В результате в каталоге `/media/bitlocker` появился виртуальный файл `dislocker-file`

Если требуется дешифровать раздел в режиме `только чтение`, надо добавить опция `-r`

```sh
$ sudo dislocker -r -V /dev/sdb1 -u%password% -- /media/bitlocker
```

И наконец монтируем раздел `/media/bitlocker`, что бы получить доступ к нашим зашифрованным файлам

```sh
$ sudo mount -o loop /media/bitlocker/dislocker-file /media/mount
```

Если требуется получить доступ к файлам в режиме `только чтение`, надо добавить опцию `-r`

```sh
$ sudo mount -r -o loop /media/bitlocker/dislocker-file /media/mount
```