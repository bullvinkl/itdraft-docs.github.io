---
title: "Расширить LVM-раздел для GPT таблиц диска на Centos 7"
date: "2019-08-19"
categories: 
  - File-System
tags: 
  - "centos"
  - "fdisk"
  - "gdisk"
  - "gpt"
  - "lvm"
image:
  path: /commons/1353848_735f_3.jpg
  alt: "Расширить LVM-раздел для GPT"
---

> **GPT** — стандарт формата размещения таблиц разделов на физическом жестком диске. Он является частью Расширяемого микропрограммного интерфейса (англ. Extensible Firmware Interface, EFI) — стандарта, предложенного Intel на смену BIOS. EFI использует GPT там, где BIOS использует Главную загрузочную запись (англ. Master Boot Record, MBR).
{: .prompt-tip }

На одном из серверов при выполнении команды `fdisk -l` вылезло предупреждение:

```sh
$ sudo fdisk -l
WARNING: fdisk GPT support is currently new, and therefore in an experimental phase. Use at your own discretion.

Disk /dev/sda: 107.4 GB, 107374182400 bytes, 209715200 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
Disk label type: gpt
Disk identifier: B0305913-98D6-4029-B711-1361097B7404

#         Start          End    Size  Type            Name
 1         2048       411647    200M  EFI System      EFI System Partition
 2       411648      2508799      1G  Microsoft basic
 3      2508800     35649535   15,8G  Linux LVM

Disk /dev/mapper/centos-root: 14.8 GB, 14814281728 bytes, 28934144 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes

Disk /dev/mapper/centos-swap: 2147 MB, 2147483648 bytes, 4194304 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
```

Гугл переводчик нам говорит следующее:  
Поддержка fdisk GPT в настоящее время является новой, и поэтому находится на экспериментальной стадии. Используйте по своему усмотрению.

Устанавливаем утилиту `gdisk`

```sh
$ sudo yum -y install gdisk
```

Посмотрим информацию о диске через эту утилиту

```sh
$ sudo gdisk -l /dev/sda
GPT fdisk (gdisk) version 0.8.10

Partition table scan:
  MBR: protective
  BSD: not present
  APM: not present
  GPT: present

Found valid GPT with protective MBR; using GPT.
Disk /dev/sda: 209715200 sectors, 100.0 GiB
Logical sector size: 512 bytes
Disk identifier (GUID): B0305913-98D6-4029-B711-1361097B7404
Partition table holds up to 128 entries
First usable sector is 34, last usable sector is 209715166
Partitions will be aligned on 2048-sector boundaries
Total free space is 174067645 sectors (83.0 GiB)

Number  Start (sector)    End (sector)  Size       Code  Name
   1            2048          411647   200.0 MiB   EF00  EFI System Partition
   2          411648         2508799   1024.0 MiB  0700
   3         2508800        35649535   15.8 GiB    8E00
```

Тут следует обратить внимание на последнюю строчку, значение столбца `End (sector) = 35649535`, это значение нам потребуется в дальнейшем

Приступим к созданию нового раздела

```sh
$ sudo gdisk /dev/sda
GPT fdisk (gdisk) version 0.8.10

Partition table scan:
  MBR: protective
  BSD: not present
  APM: not present
  GPT: present

Found valid GPT with protective MBR; using GPT.

Command (? for help): ?
b       back up GPT data to a file
c       change a partition's name
d       delete a partition
i       show detailed information on a partition
l       list known partition types
n       add a new partition
o       create a new empty GUID partition table (GPT)
p       print the partition table
q       quit without saving changes
r       recovery and transformation options (experts only)
s       sort partitions
t       change a partition's type code
v       verify disk
w       write table to disk and exit
x       extra functionality (experts only)
?       print this menu

Command (? for help): n
Partition number (4-128, default 4): 4
First sector (34-209715166, default = 35649536) or {+-}size{KMGTP}: 35649536
Last sector (35649536-209715166, default = 209715166) or {+-}size{KMGTP}: 209715166
Current type is 'Linux filesystem'
Hex code or GUID (L to show codes, Enter = 8300): L
0700 Microsoft basic data  0c01 Microsoft reserved    2700 Windows RE
3000 ONIE boot             3001 ONIE config           4100 PowerPC PReP boot
4200 Windows LDM data      4201 Windows LDM metadata  7501 IBM GPFS
7f00 ChromeOS kernel       7f01 ChromeOS root         7f02 ChromeOS reserved
8200 Linux swap            8300 Linux filesystem      8301 Linux reserved
8302 Linux /home           8400 Intel Rapid Start     8e00 Linux LVM
a500 FreeBSD disklabel     a501 FreeBSD boot          a502 FreeBSD swap
a503 FreeBSD UFS           a504 FreeBSD ZFS           a505 FreeBSD Vinum/RAID
a580 Midnight BSD data     a581 Midnight BSD boot     a582 Midnight BSD swap
a583 Midnight BSD UFS      a584 Midnight BSD ZFS      a585 Midnight BSD Vinum
a800 Apple UFS             a901 NetBSD swap           a902 NetBSD FFS
a903 NetBSD LFS            a904 NetBSD concatenated   a905 NetBSD encrypted
a906 NetBSD RAID           ab00 Apple boot            af00 Apple HFS/HFS+
af01 Apple RAID            af02 Apple RAID offline    af03 Apple label
af04 AppleTV recovery      af05 Apple Core Storage    be00 Solaris boot
bf00 Solaris root          bf01 Solaris /usr & Mac Z  bf02 Solaris swap
bf03 Solaris backup        bf04 Solaris /var          bf05 Solaris /home
bf06 Solaris alternate se  bf07 Solaris Reserved 1    bf08 Solaris Reserved 2
bf09 Solaris Reserved 3    bf0a Solaris Reserved 4    bf0b Solaris Reserved 5
c001 HP-UX data            c002 HP-UX service         ea00 Freedesktop $BOOT
eb00 Haiku BFS             ed00 Sony system partitio  ed01 Lenovo system partit
Press the <Enter> key to see more codes:
ef00 EFI System            ef01 MBR partition scheme  ef02 BIOS boot partition
fb00 VMWare VMFS           fb01 VMWare reserved       fc00 VMWare kcore crash p
fd00 Linux RAID
Hex code or GUID (L to show codes, Enter = 8300): 8e00
Changed type of partition to 'Linux LVM'

Command (? for help): w

Final checks complete. About to write GPT data. THIS WILL OVERWRITE EXISTING
PARTITIONS!!

Do you want to proceed? (Y/N): Y
OK; writing new GUID partition table (GPT) to /dev/sda.
Warning: The kernel is still using the old partition table.
The new table will be used at the next reboot.
The operation has completed successfully.
```

Где мы использовали следующие значения:

- Command (? for help): n - `Создать новый раздел`
- Partition number (4-128, default 4): 4 - `Номер раздела`
- First sector (34-209715166, default = 35649536) or {+-}size{KMGTP}: 35649536 - `Первый сектор` (про этот параметр я говорил выше, выделено)
- Last sector (35649536-209715166, default = 209715166) or {+-}size{KMGTP}: 209715166 - `Последний сектор`
- Hex code or GUID (L to show codes, Enter = 8300): 8e00 - `Код для LVM`
- Command (? for help): w - `Записать изменения`
- Do you want to proceed? (Y/N): Y - `Подтвердить`

Проинформируем операционную систему об изменении таблицы разделов

```sh
$ sudo partprobe
```

Смотрим что получилось

```sh
$ sudo gdisk -l /dev/sda
GPT fdisk (gdisk) version 0.8.10

Partition table scan:
  MBR: protective
  BSD: not present
  APM: not present
  GPT: present

Found valid GPT with protective MBR; using GPT.
Disk /dev/sda: 209715200 sectors, 100.0 GiB
Logical sector size: 512 bytes
Disk identifier (GUID): B0305913-98D6-4029-B711-1361097B7404
Partition table holds up to 128 entries
First usable sector is 34, last usable sector is 209715166
Partitions will be aligned on 2048-sector boundaries
Total free space is 2014 sectors (1007.0 KiB)

Number  Start (sector)    End (sector)  Size       Code  Name
   1            2048          411647   200.0 MiB   EF00  EFI System Partition
   2          411648         2508799   1024.0 MiB  0700
   3         2508800        35649535   15.8 GiB    8E00
   4        35649536       209715166   83.0 GiB    8E00  Linux LVM
```

Как видно, появится новый раздел `/dev/sda4`

Создадим физический том `/dev/sda4` через команду

```sh
$ sudo pvcreate /dev/sda4
  Physical volume "/dev/sda4" successfully created.
```

Ищем наш VG Name:

```sh
$ sudo vgdisplay
  --- Volume group ---
  VG Name               centos
  System ID
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  3
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                2
  Open LV               2
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               15,80 GiB
  PE Size               4,00 MiB
  Total PE              4045
  Alloc PE / Size       4044 / <15,80 GiB
  Free  PE / Size       1 / 4,00 MiB
  VG UUID               h1fisC-E0Jq-y0hg-CNsK-tbEp-FlCx-we4kc0
```

Расширяем том `centos`

```sh
$ sudo vgextend centos /dev/sda4
  Volume group "centos" successfully extended
```

Проверяем что получилось

```sh
$ sudo pvscan
  PV /dev/sda3   VG centos          lvm2 [15,80 GiB / 4,00 MiB free]
  PV /dev/sda4   VG centos          lvm2 [<83,00 GiB / <83,00 GiB free]
  Total: 2 [<98,80 GiB] / in use: 2 [<98,80 GiB] / in no VG: 0 [0   ]
```

Ищем наш LV Path:

```sh
$ sudo lvdisplay
  --- Logical volume ---
  LV Path                /dev/centos/root
  LV Name                root
  VG Name                centos
  LV UUID                2Js7Fd-vndx-X1e8-mOGt-EgX6-CRn3-jrBeMB
  LV Write Access        read/write
  LV Creation host, time localhost, 2019-07-10 10:32:35 +0300
  LV Status              available
  # open                 1
  LV Size                <13,80 GiB
  Current LE             3532
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:0

  --- Logical volume ---
  LV Path                /dev/centos/swap
  LV Name                swap
  VG Name                centos
  LV UUID                hGtREN-VZpe-djLb-rT7g-drKz-64E5-mh4Dtd
  LV Write Access        read/write
  LV Creation host, time tocalhost, 2019-07-10 10:32:36 +0300
  LV Status              available
  # open                 2
  LV Size                2,00 GiB
  Current LE             512
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:1
```

Расширяем логический том `/dev/centos/root` и активируем его

```sh
$ sudo lvextend -l +100%FREE /dev/centos/root -r
  Size of logical volume centos/root changed from <13,80 GiB (3532 extents) to <96,80 GiB (24780 extents).
  Logical volume centos/root successfully resized.
meta-data=/dev/mapper/centos-root isize=512    agcount=4, agsize=904192 blks
         =                       sectsz=4096  attr=2, projid32bit=1
         =                       crc=1        finobt=0 spinodes=0
data     =                       bsize=4096   blocks=3616768, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal               bsize=4096   blocks=2560, version=2
         =                       sectsz=4096  sunit=1 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
data blocks changed from 3616768 to 25374720
```

Проверяем через команду `gdisk -l /dev/sda`

```sh
$ sudo gdisk -l /dev/sda
GPT fdisk (gdisk) version 0.8.10

Partition table scan:
  MBR: protective
  BSD: not present
  APM: not present
  GPT: present

Found valid GPT with protective MBR; using GPT.
Disk /dev/sda: 209715200 sectors, 100.0 GiB
Logical sector size: 512 bytes
Disk identifier (GUID): B0305913-98D6-4029-B711-1361097B7404
Partition table holds up to 128 entries
First usable sector is 34, last usable sector is 209715166
Partitions will be aligned on 2048-sector boundaries
Total free space is 2014 sectors (1007.0 KiB)

Number  Start (sector)    End (sector)  Size       Code  Name
   1            2048          411647   200.0 MiB   EF00  EFI System Partition
   2          411648         2508799   1024.0 MiB  0700
   3         2508800        35649535   15.8 GiB    8E00
   4        35649536       209715166   83.0 GiB    8E00  Linux LVM
```

Проверяем через команду `df -h`

```sh
$ sudo df -h
Файловая система        Размер Использовано  Дост Использовано% Cмонтировано в
/dev/mapper/centos-root    97G         1,3G   96G            2% /
devtmpfs                  858M            0  858M            0% /dev
tmpfs                     870M            0  870M            0% /dev/shm
tmpfs                     870M         9,3M  860M            2% /run
tmpfs                     870M            0  870M            0% /sys/fs/cgroup
/dev/sda2                1014M         176M  839M           18% /boot
/dev/sda1                 200M          12M  189M            6% /boot/efi
tmpfs                     174M            0  174M            0% /run/user/0
```
