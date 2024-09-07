---
title: "Автоматический бэкап файлов при подключении съемного носителя"
date: "2018-12-26"
categories: 
  - Automation
tags: 
  - "backup"
  - "bash"
  - "ubuntu"
image:
  path: /commons/1039062_e553_4.jpg
  alt: "Автоматический бэкап файлов при подключении съемного носителя"
---

> **Резервное копирование** — процесс создания копии данных на носителе (жёстком диске, дискете и т. д.), предназначенном для восстановления данных в оригинальном или новом месте их расположения в случае их повреждения или разрушения.
{: .prompt-tip }

В этой статье вы узнаете, как выполнять автоматическое резервное копирование данных на съемный носитель после его подключения к компьютеру с Linux. Данный метод резервного копирования протестирован на USB flash-карте.

## Подготовительные действия

Вначале нам нужно предоставить `udev` атрибуты съемного носителя, которые будут использоваться для резервного копирования. Подключим флэшку к работающей системе и выполним команду `lsusb`, чтобы определить его ID.

```sh
$ lsusb
Bus 002 Device 017: ID 046d:c062 Logitech, Inc. M-UAS144 [LS1 Laser Mouse]
Bus 002 Device 003: ID 058f:6362 Alcor Micro Corp. Flash Card Reader/Writer
Bus 002 Device 020: ID 13fe:6300 Kingston Technology Company Inc. 
Bus 002 Device 002: ID 8087:0024 Intel Corp. Integrated Rate Matching Hub
Bus 002 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
Bus 001 Device 002: ID 8087:0024 Intel Corp. Integrated Rate Matching Hub
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
```

В данном случае `Kingston Technology Company Inc` - и есть наша флэшка, ее `ID`: `13fe`

Отключаем флэшку, и создаем файл правил `udev` `10.autobackup.rules` в директории `/etc/udev/rules.d/`  
Цифра 10 в имени файла указывает порядок выполнения правил

```sh
$ sudo nano /etc/udev/rules.d/10.autobackup.rules
SUBSYSTEM=="block", ACTION=="add", ATTRS{idVendor}=="13fe" SYMLINK+="external%n", RUN+="/bin/usb_backup.sh"
```

## Скрипт бэкапирования

Теперь создадим скрипт резервного копирования, который будет автоматически создавать резервные копии файлов на съемный USB при подключении к системе.

```sh
$ sudo nano /bin/usb_backup.sh 
#!/usr/bin/bash
BACKUP_SOURCE="/home/user/important"
BACKUP_DEVICE="/dev/external1"
MOUNT_POINT="/mnt/external"

#проверяем, есть ли каталог для монтирования, если нет создаем
if [ ! -d “MOUNT_POINT” ] ; then 
	/bin/mkdir  “$MOUNT_POINT”; 
fi

/bin/mount  -t  auto  “$BACKUP_DEVICE”  “$MOUNT_POINT”

#запускаем процесс бэкапирования
/usr/bin/rsync -auz  "$MOUNT_LOC" "$BACKUP_SOURCE" && /bin/umount "$BACKUP_DEVICE"
exit
```

Делаем скрипт исполняемым и перезагружаем правила `udev`

```sh
$ sudo chmod +x /bin/usb_backup.sh
$ udevadm control --reload
```

Теперь при следующем подключении внешнего жесткого диска или любого другого устройства, которое вы настроили, все ваши документы из указанного места будут автоматически скопированы.
