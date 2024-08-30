---
layout: post
title: "[Решено] Установка Python 3.12 в Rocky Linux 9 / AlmaLinux и назначаем его для использования по умолчанию"
# description: ""
date: 2024-01-18 18:40 +0300
categories:
  - Linux
  - Python
tags:
  - python
  - almalinux
  - centos
  - linux
  - rocky linux
image:
  path: /commons/scale_2400.png
  alt: "Установка Python 3.12"
---

> Python — высокоуровневый язык программирования общего назначения с динамической строгой типизацией и автоматическим управлением памятью, ориентированный на повышение производительности разработчика, читаемости кода и его качества, а также на обеспечение переносимости написанных на нём программ.

Обновляем ОС и устанавливаем необходимые пакеты
```sh
$ sudo dnf -y update
$ sudo dnf -y install gcc openssl-devel bzip2-devel libffi-devel zlib-devel wget make tar
```

Скачиваем и распаковываем дистрибутив Python с официального сайта
```sh
$ sudo wget https://www.python.org/ftp/python/3.12.1/Python-3.12.1.tgz
$ sudo tar -xf Python-3.12.1.tgz
```

Переходим в каталог, куда распаковался дистрибутив Python, и запускаем процесс конфигурирования
```sh
$ cd Python-3.12.1
$ sudo ./configure --enable-optimizations
```

Смотрим количество ядер на ВМ и выставляем такое же количество потоков для сборки ПО

```sh
$ sudo nproc
2

$ sudo make -j 2
```

Запускаем установку ПО
```sh
$ sudo make altinstall
```

Назначаем установленную версию Python для использования по умолчанию
```sh
$ sudo update-alternatives --install /usr/bin/python python /usr/local/bin/python3.12 20
$ sudo update-alternatives --install /usr/bin/python3 python3 /usr/local/bin/python3.12 20
```