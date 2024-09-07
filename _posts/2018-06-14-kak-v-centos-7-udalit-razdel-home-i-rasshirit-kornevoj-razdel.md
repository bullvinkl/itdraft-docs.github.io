---
title: "Как в CentOS 7 удалить раздел /home и расширить корневой раздел"
date: "2018-06-14"
categories: 
  - File-System
tags: 
  - "centos"
  - "lvextend"
  - "lvremove"
  - "unmount"
image:
  path: /commons/1377042_ba6b_13.jpg
  alt: "Удалить раздел /home и расширить корневой раздел"
---

> В Unix-подобных ОС, включая Linux и macOS, раздел /home является каталогом для хранения файлов и настроек пользователей. Он обычно располагается на отдельном разделе жёсткого диска и монтируется в /home при запуске системы.
{: .prompt-tip }

Смотрим разделы:

```sh
$ df -h
Файловая система        Размер Использовано  Дост Использовано% Cмонтировано в
/dev/mapper/centos-root    36G         899M   35G            3% /
devtmpfs                  486M            0  486M            0% /dev
tmpfs                     497M            0  497M            0% /dev/shm
tmpfs                     497M         6,6M  490M            2% /run
tmpfs                     497M            0  497M            0% /sys/fs/cgroup
/dev/sda1                1014M         125M  890M           13% /boot
/dev/mapper/centos-home    18G          33M   18G            1% /home
tmpfs                     100M            0  100M            0% /run/user/0
```

Размантируем раздел `/home`, иначе дальнейшие действия не выполнятся

```sh
$ sudo umount /home
```

Удаляем раздел `/home`:

```sh
$ sudo lvremove /dev/mapper/centos-home
Do you really want to remove active logical volume centos/home? [y/n]: y
  Logical volume "home" successfully removed
```

Расширяем корневой раздел:

```sh
$ sudo lvextend -l +100%FREE -r /dev/mapper/centos-root
```

Не забываем закоментировать или удалить строку монтирования из файла `/etc/fstab`, иначе после перезагрузки, ОС не загрузится:

```sh
$ sudo cat /etc/fstab
#
# /etc/fstab
# Created by anaconda on Thu Jun 14 10:36:47 2018
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
/dev/mapper/centos-root /                       xfs     defaults        0 0
UUID=57c0960d-c69c-42e7-80bc-84c7fc57ba41 /boot                   xfs     defaults        0 0
#/dev/mapper/centos-home /home                   xfs     defaults        0 0
/dev/mapper/centos-swap swap                    swap    defaults        0 0
```

Смотрим на результат:

```sh
$ sudo df -h
Файловая система        Размер Использовано  Дост Использовано% Cмонтировано в
/dev/mapper/centos-root    53G         899M   52G            2% /
devtmpfs                  486M            0  486M            0% /dev
tmpfs                     497M            0  497M            0% /dev/shm
tmpfs                     497M         6,6M  490M            2% /run
tmpfs                     497M            0  497M            0% /sys/fs/cgroup
/dev/sda1                1014M         125M  890M           13% /boot
tmpfs                     100M            0  100M            0% /run/user/0
```
