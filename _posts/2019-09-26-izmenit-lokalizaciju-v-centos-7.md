---
title: "Изменить локализацию в CentOS 7"
date: "2019-09-26"
categories: 
  - Linux
tags: 
  - "centos"
  - "locale"
  - "localectl"
image:
  path: /commons/1003248_1dd0_2.jpg
  alt: "Изменить локализацию"
---

>**localectl** - это утилита для управления локальной конфигурацией и ресурсами на компьютере, позволяющая изменять параметры локализации, часовой пояс и язык. Она обычно используется в операционных системах на основе Linux, включая Ubuntu и другие дистрибутивы.
{: .prompt-tip }

Смотрим текущий язык

```sh
$ sudo localectl
System Locale: n/a
    VC Keymap: n/a
   X11 Layout: n/a
```

Смотрим, доступен ли русский язык

```sh
$ sudo localectl list-locales | grep ru
ru_RU
ru_RU.iso88595
ru_RU.koi8r
ru_RU.utf8
ru_UA
ru_UA.koi8u
ru_UA.utf8
russian
```

Задаем кодировку UTF-8 в консоли CentOS 7 и выбрать английский язык в качестве системного

```sh
$ sudo localectl set-locale LANG=en_US.UTF-8
```

Перезагружаем сервер, проверяем настройки.

```sh
$ sudo reboot
[root@localhost]# localectl status
System Locale: LANG=en_US.UTF-8
    VC Keymap: us
   X11 Layout: us,ru
  X11 Variant: ,
  X11 Options: grp:alt_shift_toggle
```

Установить русский язык в качестве системного

```sh
$ sudo localectl set-locale LANG=ru_RU.UTF-8
```

Посмотреть доступные раскладки русских клавиатур

```sh
$ sudo localectl list-keymaps | grep ru
ruwin_alt-CP1251
ruwin_alt-KOI8-R
ruwin_alt-UTF-8
ruwin_alt_sh-UTF-8
ruwin_cplk-CP1251
ruwin_cplk-KOI8-R
ruwin_cplk-UTF-8
ruwin_ct_sh-CP1251
ruwin_ct_sh-KOI8-R
ruwin_ct_sh-UTF-8
ruwin_ctrl-CP1251
ruwin_ctrl-KOI8-R
ruwin_ctrl-UTF-8
```

Установить русскую раскладку с переключением по `ALT+SHIFT`

```sh
$ sudo localectl set-keymap ruwin_alt_sh-UTF-8
```

После применения необходимо перезагрузить сервер

```sh
$ sudo reboot
```
