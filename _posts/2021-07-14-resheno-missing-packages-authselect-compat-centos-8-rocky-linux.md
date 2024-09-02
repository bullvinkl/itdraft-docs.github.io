---
title: "[РЕШЕНО] missing packages: authselect-compat / Centos 8 / Rocky Linux"
date: "2021-07-14"
categories: 
  - Linux
  - Kickstart
tags: 
  - "centos"
  - "kickstart"
  - "rocky-linux"
image:
  path: /commons/warning02.jpg
  alt: "missing packages: authselect-compat"
---

## Ошибка

При установке Centos 8 / Rocky linux 8 при помощи kickstart из минимального образа (`CentOS-8.3.2011-x86_64-minimal.iso`, `Rocky-8.4-x86_64-minimal.iso`) появляется ошибка:

```
missing packages: authselect-compat
```

Причина этой ошибки в том, что пакет `authselect-compat` лежит в репозитории AppStream, а в минимальном образе дистрибутива этот репозиторий не подключен

## Решение

В kickstart-файл прописать опции для добавления репозиториев (например для Rocky Linux):

```
...
# Create repositories
repo --name=BaseOS --baseurl=https://download.rockylinux.org/pub/rocky/8/BaseOS/x86_64/os/
repo --name=AppStream --baseurl=https://download.rockylinux.org/pub/rocky/8/AppStream/x86_64/os/
...
```

Эти строки приписываются например после опций разбивки дисков