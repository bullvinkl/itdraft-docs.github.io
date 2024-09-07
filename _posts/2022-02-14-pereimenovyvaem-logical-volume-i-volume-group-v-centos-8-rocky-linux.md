---
title: "Переименовываем Logical Volume и Volume Group в Centos 8 / Rocky Linux"
date: "2022-02-14"
categories: 
  - Linux
  - Disk
tags: 
  - "centos"
  - "linux"
  - "lv"
  - "lvm"
  - "lvrename"
  - "pv"
  - "rocky-linux"
  - "vg"
  - "vgrename"
image:
  path: /commons/1.png
  alt: "Переименовываем Logical Volume и Volume Group"
---

> **Менеджер логических томов (logical volume manager)** — подсистема операционных систем Linux и OS/2, позволяющая использовать разные области одного жёсткого диска и/или области с разных жёстких дисков как один логический том. Реализована с помощью подсистемы device mapper.  
> PV (Physical Volume) — физические тома  
> VG (Volume Group) — группа томов  
> LV (Logical Volume) — логические разделы
{: .prompt-tip }

## Переименовываем Logical Volume

Команды

```sh
$ sudo lvrename /dev/vg02/lvold /dev/vg02/lvnew
$ sudo lvrename vg02 lvold lvnew
```

Смотрим текущие параметры Logical Volume и Volume Group

```sh
$ sudo lvs
  LV   VG     Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  root centos -wi-ao---- <7.00g                                                
  swap centos -wi-ao---- 512.00m
```

Переименовываем

```sh
$ sudo lvrename centos root rootnew
  Renamed "root" to "rootnew" in volume group "centos"
```

Проверяем

```sh
$ sudo lvs
  LV      VG     Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  rootnew centos -wi-ao---- <7.00g                                             
  swap    centos -wi-ao---- 512.00m 
```

> Внимание, если на данном этапе перезагрузить ВМ, она не загрузится
{: .prompt-danger }

Редактируем файл fstab

```sh
$ sudo nano /etc/fstab
...
/dev/mapper/centos-rootnew /                       xfs     defaults        0 0
```

Редактируем grub для GPT (UEFI-based) систем

```sh
$ sudo nano /etc/default/grub
GRUB_CMDLINE_LINUX="crashkernel=auto resume=/dev/mapper/centos-swap rd.lvm.lv=centos/rootnew rd.lvm.lv=centos/swap rhgb quiet"
```

Перезагружаемся

> Перед загрузкой ОС выбираем первую строку, жмем "e" и правим загрузочную запись, указывая верный Lovical Volume / Volume Group.  
> Жмем ctrl+x для загрузки.
{: .prompt-info }

![](/assets/img/posts/2022/02/14/lv1.png){: w="300" }

![](/assets/img/posts/2022/02/14/lv2.png){: w="300" }

Создаем новый grub.cfg файл для GPT (UEFI-based) систем

```sh
$ sudo grub2-mkconfig -o /boot/efi/EFI/redhat/grub.cfg	# для Centos

$ sudo grub2-mkconfig -o /boot/efi/EFI/rocky/grub.cfg  # для Rocky Linux
```

Теперь можно перезагружаться, запись grub корректна

```sh
$ sudo reboot
```

## Переименовываем Volume Group

Команды

```sh
$ sudo vgrename /dev/vg02 /dev/my_volume_group
$ sudo vgrename vg02 my_volume_group
```

Смотрим текущие параметры Volume Group

```sh
$ sudo vgs
  VG     #PV #LV #SN Attr   VSize  VFree
  centos   1   2   0 wz--n- <7.50g    0
```

Переименовываем

```sh
$ sudo vgrename centos rocky
 Volume group "centos" successfully renamed to "rocky"
```

Проверяем

```sh
$ sudo lvs
  VG    #PV #LV #SN Attr   VSize  VFree
  rocky   1   2   0 wz--n- <7.50g    0
```

> Внимание, если на данном этапе перезагрузить ВМ, она не загрузится
{: .prompt-danger }

Редактируем файл fstab

```sh
$ sudo nano /etc/fstab
...
/dev/mapper/rocky-rootnew /                       xfs     defaults        0 0
...
/dev/mapper/rocky-swap none                    swap    defaults        0 0
```

Редактируем grub для GPT (UEFI-based) систем

```sh
$ sudo nano /etc/default/grub
GRUB_CMDLINE_LINUX="crashkernel=auto resume=/dev/mapper/rocky-swap rd.lvm.lv=rocky/rootnew rd.lvm.lv=rocky/swap rhgb quiet"
```

Перезагружаемся

> Перед загрузкой ОС выбираем первую строку, жмем "e" и правим загрузочную запись, указывая верный Lovical Volume / Volume Group.
> Жмем ctrl+x для загрузки.
{: .prompt-info }

![](/assets/img/posts/2022/02/14/lv1.png){: w="300" }

![](/assets/img/posts/2022/02/14/lv2.png){: w="300" }

Создаем новый grub.cfg файл для GPT (UEFI-based) систем

```sh
$ sudo grub2-mkconfig -o /boot/efi/EFI/redhat/grub.cfg	# для Centos

$ sudo grub2-mkconfig -o /boot/efi/EFI/rocky/grub.cfg  # для Rocky Linux
```

Теперь можно перезагружаться, запись grub корректна

```sh
$ sudo reboot
```
