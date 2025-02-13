---
layout: post
title: "Меняем swap область на swap файл"
date: "2024-09-17"
categories:
  - File-System
tags:
  - swap
  - debian
  - astra
image:
  path: /commons/laptops.png
  alt: "Меняем swap область на swap файл"
---

> **Swap** (своп) - это механизм виртуальной памяти в операционной системе Linux, который позволяет оперативной памяти (ОЗУ) компенсировать нехватку физической памяти. Когда ОЗУ заполнена и не хватает места для новых данных, Linux переносит неиспользуемые страницы памяти (например, файлы, которые не используются в данный момент) на специальный раздел на жёстком диске или файловой системе, называемый областью подкачки (swap area или swap space).
{: .prompt-tip }

Для чего мы это делаем?
- Со swap файлом гораздо удобней работать, если требуется его расширить или уменьшить.
- Если по какой-либо причине корневой диск забился, мы можем отключить swap, и у нас появляется дополнительное дисковое пространство.

Менять swap область на swap файл будем на примере ОС Astra Linux

Отключаем swap

```bash
$ sudo swapoff -a
$ sudo nano /etc/fstab
...
#/dev/mapper/astra--vg-swap_1 none            swap    sw              0       0
```

Удаляем swap область, для этого удаляем раздел `/dev/astra-vg/swap_1` и расширяем корневой раздел `/dev/astra-vg/root` (файловая система `ext4`)

```bash
$ sudo lvdisplay
$ sudo lvremove /dev/astra-vg/swap_1
$ sudo lvextend -l +100%FREE /dev/astra-vg/root
$ sudo resize2fs /dev/astra-vg/root
```

Вносим изменения в файл `resume`
```bash
$ sudo nano /etc/initramfs-tools/conf.d/resume
#RESUME=/dev/mapper/astra--vg-swap_1
RESUME=none
```

Обновляем параметры загрузчика и обновляем `initramfs` после внесения изменений

```bash
$ sudo update-grub
$ sudo update-initramfs -u
```

Создаем swap файл и монтируем его

```bash
$ sudo fallocate -l 12G /swapfile
$ sudo chmod 600 /swapfile
$ sudo mkswap /swapfile
$ sudo swapon /swapfile
$ echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab > /dev/null 2>&1
```

## UPD 2025-02-13

В Centos 7 во время выделения области появилась ошибка
> swapon: /swapfile: swapon failed: Invalid argument
{: .prompt-warning }

Решение: вместо `fallocate` использовать `dd`
```bash
$ sudo dd if=/dev/zero of=/swapfile bs=1MiB count=4000
```
