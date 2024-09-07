---
title: "Добавить Swap в CentOS"
date: "2018-05-16"
categories: 
  - Linux
tags: 
  - "centos"
  - "swap"
image:
  path: /commons/1334578_287d_2.jpg
  alt: "Добавить Swap"
---

> **Swap** (своп) - это механизм виртуальной памяти в операционной системе Linux, который позволяет оперативной памяти (ОЗУ) компенсировать нехватку физических ресурсов. Swap - это раздел или файл на диске, предназначенный для хранения данных из ОЗУ, когда система испытывает недостаток в физической памяти.
{: .prompt-tip }

Смотрим, какого размера swap

```sh
$ sudo free -m
```

либо

```sh
$ sudo swapon -s
```


Создаем файл подкачки

```sh
$ sudo dd if=/dev/zero of=/swap.img bs=1M count=12288
```

или

```sh
$ sudo fallocate -l 12G /swap.img
```

Проверяем

```sh
$ sudo ls -alh /swap.img
```

Назначаем права

```sh
$ sudo chmod 600 /swap.img
```

Создаем пространство подкачки

```sh
$ sudo mkswap /swap.img
```

Включаем файл подкачки

```sh
$ sudo swapon /swap.img
```

Проверяем

```sh
$ sudo swapon -s
```

Добавляем файл подкачки в автозагрузку (`/etc/fstab`)

```sh
$ sudo nano /etc/fstab
...
/swap.img swap swap defaults 0 0
```

Чтобы выключить этот файл подкачки

```sh
$ sudo swapoff /swap.img
```

Проверяем

```sh
$ sudo swapon -s
```
