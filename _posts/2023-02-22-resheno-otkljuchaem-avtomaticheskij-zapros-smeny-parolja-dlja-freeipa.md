---
title: "[Решено] Отключаем автоматический запрос смены пароля для FreeIPA"
date: "2023-02-22"
categories: 
  - Linux
  - FreeIPA
tags: 
  - "centos"
  - "freeipa"
  - "linux"
  - "rocky-linux"
image:
  path: /commons/hack_b-min-1.png
  alt: "Отключаем автоматический запрос смены пароля для FreeIPA"
---

Во FreeIPA при добавлении пользователя и последующей его авторизации (в админке FreeIPA или по SSH) запрашивается принудительная смена пароля. Иногда возникают ситуации, когда надо отключить принудительную смену пароля.

Это происходит из-за того, что параметр krbPasswordExpiration по умолчанию равен дате создания пользователя.

Авторизуемся в домене и изменим параметр krbPasswordExpiration

```sh
$ kinit admin
$ ipa user-mod m.makarov --setattr=krbPasswordExpiration=20230817011529Z
```