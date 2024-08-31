---
title: "[Решено] Расширить корневой LVM-раздел. Еще один способ"
date: "2023-07-14"
categories: 
  - Linux
  - Disk
tags: 
  - ext4
  - fdisk
  - linux
  - lvm
  - xfs
image:
  path: /commons/computer.jpg
  alt: "Расширить корневой LVM-раздел"
---

> **Менеджер логических томов** — подсистема операционных систем Linux и OS/2, позволяющая использовать разные области одного жёсткого диска и/или области с разных жёстких дисков как один логический том. Реализована с помощью подсистемы device mapper. 

Исходные данные:
- ОС: Debian 11
- Тип разделов: MBR
- Файловая система: XFS

![](/assets/img/posts/2024/07/14/image-1.png){: w="300" }
_Дисковое пространство_

![](/assets/img/posts/2024/07/14/image.png){: w="300" }
_Структура_

Как видно из разметки, диск SDA увеличили.

При попытке расширить корневой раздел через growpart появлялась ошибка: FAILED: failed to resize
![](/assets/img/posts/2024/07/14/image-2.png){: w="300" }
_FAILED: failed to resize_

Будем расширять диск через fdisk

Добавляем раздел sda3, меняем тип по LVM
```sh
$ sudo fdisk /dev/sda
Command (m for help): n (новый раздел)
Select (default p): p (раздел будет primary)
Partition number (3,4, default 3): 3 (номер раздела 3)
First sector (35649536-67108863, default 35649536): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (35649536-67108863, default 67108863): 
Command (m for help): t (изменяем тип вновь созданного раздела)
Partition number (1-3,5, default 5): 3 (номер нашего нового раздела)
Hex code or alias (type L to list all): 8e (тип раздела Linux LVM)
Command (m for help): w (сохранить изменения в таблице разделов и закрыть fdisk)
The partition table has been altered.
Syncing disks.
```

Смотрим результат
![](/assets/img/posts/2024/07/14/image-3.png){: w="300" }
_Результат_

Инициализируем раздел в качестве LVM, добавляем в группу debian и расширяем пространство
```sh
$ sudo pvcreate /dev/sda3
$ sudo vgextend debian /dev/sda3
$ sudo lvextend -l +100%FREE /dev/mapper/debian-root
```

Расширяем файловую систему
```sh
$ sudo resize2fs /dev/mapper/debian-root  # для ext4
$ sudo xfs_growfs  /dev/mapper/debian-root  # для xfs
```

Результат:
![](/assets/img/posts/2024/07/14/image-4.png){: w="300" }
_Результат_