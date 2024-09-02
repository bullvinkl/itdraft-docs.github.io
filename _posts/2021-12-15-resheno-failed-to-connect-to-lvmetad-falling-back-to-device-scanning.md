---
title: "[РЕШЕНО] Failed to connect to lvmetad. Falling back to device scanning"
date: "2021-12-15"
categories: 
  - Linux
  - Disk
tags: 
  - "debian"
  - "failed"
  - "lvm"
  - "lvmetad"
  - "resheno"
image:
  path: /commons/warning02.jpg
  alt: "Failed to connect to lvmetad"
---

При добавлении LVM-раздела в Debian 9, после перезагрузки ОС не смогла штатно загрузиться

![](/assets/img/posts/2021/12/15/img.png){: w="300" }
_WARNING : Failed to connect to lvmetad_

Что бы ОС загружалась штатно, отредактируем файл `/etc/lvm/lvm.conf`

```sh
$ sudo nano /etc/lvm/lvm.conf
...
    use_lvmetad = 0
...
```

Обновим initramfs для текущего ядра ОС (от root-пользователя) и перезагрузимся

```sh
$ sudo su
# update-initramfs -k $(uname -r) -u; sync
# reboot
```