---
title: "[РЕШЕНО] Меняем локализацию в Debian 8 - 11"
date: "2021-08-31"
categories: 
  - Linux
  - Debian
tags: 
  - "debian"
  - "locale"
image:
  path: /commons/no-release-repo.jpg
  alt: "Меняем локализацию в Debian"
---

> **locale** — UNIX‐утилита, выводящая информацию о региональных настройках операционной системы

Смотрим текущую локализацию

```sh
$ locale -a
C
C.UTF-8
POSIX
ru_RU.utf8
```

Запускаем утилиту, для переконфигурирования локализации

```sh
$ sudo dpkg-reconfigure locales
```

Выбираем необходимые локализации: `en_US.UTF-8`, `ru_RU.UTF-8`

![](/assets/img/posts/2021/08/31/image-5.png){: w="300" }

Выбираем локализацию по умолчанию: `en_US.UTF-8`

![](/assets/img/posts/2021/08/31/image-6.png){: w="300" }

Проверяем

```sh
$ locale -a
C
C.UTF-8
en_US.utf8
POSIX
ru_RU.utf8
```