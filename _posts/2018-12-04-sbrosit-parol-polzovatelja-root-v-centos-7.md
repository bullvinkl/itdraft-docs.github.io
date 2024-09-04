---
title: "Сбросить пароль пользователя Root в Centos 7"
date: "2018-12-04"
categories: 
  - Linux
tags: 
  - "centos"
  - "grub"
  - "password"
  - "reset"
  - "root"
image:
  path: /commons/cybersec_digest_3.png
  alt: "Сбросить пароль пользователя Root в Centos"
---

> **ROOT** - это учетная запись главного администратора или суперпользователя в UNIX-подобных системах, включая операционную систему Android. Пользователь с правами ROOT имеет полный доступ к системе и может выполнять любые операции, включая изменение файлов, настройку параметров и управление ресурсами.

Для того, чтобы сбросить пароль пользователя `root` необходимо во время загрузки меню `grub` нажать клавишу `e` на клавиатуре

![](/assets/img/posts/2018/12/04/wp_centos7-forgot-root-password.png){: w="300" }
_grub menu_

Затем надо прокрутить список и найти строчку `linux16 /vmlinz-3.10.0-123.6.3.el7.x86_64 root=/dev/mapper/centos-root ro rd.lvm.lv=centos/swap ...` в которой `ro` заменить на `rw init=/sysroot/bin/sh`

![](/assets/img/posts/2018/12/04/wp_centos7-forgot-root-password-1.png){: w="300" }
_ro_

![](/assets/img/posts/2018/12/04/wp_centos7-forgot-root-password-2.png){: w="300" }
_rw init=/sysroot/bin/sh_

Нажать `Ctrl+X` на клавиатуре, чтобы начать работать в однопользовательском режим, используя оболочку `bash`. В этом режиме мы изменим пароль пользователя `root`.

![](/assets/img/posts/2018/12/04/wp_centos7-forgot-root-password-2.5.png){: w="300" }
_Ctrl+X_

В однопользовательском режиме запустите команду

```sh
:/# chroot /sysroot
```

![](/assets/img/posts/2018/12/04/wp_centos7-forgot-root-password-3.png){: w="300" }
_chroot /sysroot_

Сбросим пароль пользователя root

```sh
:/# passwd root
```

Нам будет предложено создать и подтвердить новый пароль. После создания пароля надо выполнить команду, указанную ниже, чтобы обновить параметры SELinux

```sh
:/# touch /.autorelabel
```

Далее выходим и перезагружаемся

```sh
:/# exit
:/# reboot
```