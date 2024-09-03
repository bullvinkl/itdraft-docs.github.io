---
title: "Обновление CentOS 7 до CentOS 8"
date: "2020-08-04"
categories: 
  - Linux
tags: 
  - "centos"
  - "upgrade"
image:
  path: /commons/1051518_28b0_3.jpg
  alt: "Обновление CentOS 7 > 8"
---

> **CentOS** — дистрибутив Linux, основанный на коммерческом Red Hat Enterprise Linux компании Red Hat и совместимый с ним. Согласно жизненному циклу Red Hat Enterprise Linux (RHEL), CentOS 5, 6 и 7 будут поддерживаться «до 10 лет», поскольку они основаны на RHEL. Ранее версия CentOS 4 поддерживалась семь лет.

## Подготовка

Добавляем репозиторий EPEL

```sh
$ sudo yum -y install epel-release
```

Устанавливаем утилиту `yum-utils`

```sh
$ sudo yum -y install yum-utils
```

Устанавливаем утилиту `rpmconf`

```sh
$ sudo yum -y install rpmconf
```

Выполняем проверку и сравнение конфигов

```sh
$ sudo rpmconf -a
```

После выполнения команды смотрим вывод утилиты и отвечаем на вопросы о том, какой конфиг нам нужен (текущий, дефолтный из пакета …)

Смотрим, какие у нас установлены пакеты не из репозиториев, есть ли в системе пакеты, которые можно удалить

```sh
$ sudo package-cleanup --leaves
$ package-cleanup --orphans
```

## Обновление Centos до версии 8

Установим менеджер пакетов dnf, который используется по умолчанию в CentOS 8

```sh
$ sudo yum -y install dnf
```

Удалим менеджер пакетов yum (если он в дальнейшем вам не нужен)

```sh
$ sudo dnf -y remove yum yum-metadata-parser
$ sudo rm -Rf /etc/yum
```

Обновляем Centos

```sh
$ sudo dnf -y upgrade
```

Устанавливаем необходимые пакеты для CentOS 8

```sh
$ sudo dnf -y install \
   http://mirror.centos.org/centos/8/BaseOS/x86_64/os/Packages/centos-repos-8.2-2.2004.0.1.el8.x86_64.rpm \
   http://mirror.centos.org/centos/8/BaseOS/x86_64/os/Packages/centos-release-8.2-2.2004.0.1.el8.x86_64.rpm \
   http://mirror.centos.org/centos/8/BaseOS/x86_64/os/Packages/centos-gpg-keys-8.2-2.2004.0.1.el8.noarch.rpm
```

Обновляем репозиторий EPEL

```sh
$ sudo dnf -y upgrade https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
```

Удаляем временные файлы

```sh
$ sudo dnf clean all
```

Удаляем старые ядра от Centos 7

```sh
$ sudo rpm -e `rpm -q kernel`
```

Удаляем пакеты, которые могут конфликтовать

```sh
$ sudo rpm -e --nodeps sysvinit-tools
```

Запускаем обновление системы

```sh
$ sudo dnf -y --releasever=8 --allowerasing --setopt=deltarpm=false distro-sync
```

> На этом моменте у меня возникла ошибка зависимостей
{: .prompt-warning }

```
python3-rpmconf-1.0.21-1.el8.noarch conflicts with file from package python36-rpmconf-1.0.22-1.el7.noarch
```

Решение:

```sh
$ sudo dnf -y remove python36-rpmconf
```

## Ядро для Centos 8

Устанавливаем новое ядро для CentOS 8
sh
```
$ sudo dnf -y install kernel-core
```

Устанавливаем минимальный набор пакетов через групповое управление

```sh
$ sudo dnf -y groupupdate "Core" "Minimal Install"
```

Проверяем, какая версия centos установилась

```sh
$ cat /etc/*release
CentOS Linux release 8.2.2004 (Core) 
NAME="CentOS Linux"
VERSION="8 (Core)"
ID="centos"
ID_LIKE="rhel fedora"
VERSION_ID="8"
PLATFORM_ID="platform:el8"
PRETTY_NAME="CentOS Linux 8 (Core)"
ANSI_COLOR="0;31"
CPE_NAME="cpe:/o:centos:centos:8"
HOME_URL="https://www.centos.org/"
BUG_REPORT_URL="https://bugs.centos.org/"

CENTOS_MANTISBT_PROJECT="CentOS-8"
CENTOS_MANTISBT_PROJECT_VERSION="8"
REDHAT_SUPPORT_PRODUCT="centos"
REDHAT_SUPPORT_PRODUCT_VERSION="8"

CentOS Linux release 8.2.2004 (Core) 
CentOS Linux release 8.2.2004 (Core) 
```

Удаляем временные файлы

```sh
$ sudo dnf clean all
```

## Ошибка при установке YUM

При установке возникла ошибка

```sh
...
Error: Transaction failed
```

Решение

```sh
$ cd /usr/bin
$ sudo ln -s dnf-3 yum
$ cd /etc/yum
$ sudo rm -r *
$ sudo dnf -y install yum
```