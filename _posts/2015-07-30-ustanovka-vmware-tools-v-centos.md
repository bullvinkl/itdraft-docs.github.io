---
title: "Установка VMware tools в CentOS"
date: "2015-07-30"
categories: 
  - Manuals
tags: 
  - "vmware-tool"
image:
  path: /commons/stress-min-1.png
  alt: "Установка VMware tools"
---

> **VMware Tools** - это набор утилит, разработанный VMware, который улучшает взаимодействие виртуальной машины и облачной платформы, увеличивает производительность операционной системы в виртуальной машине, а также улучшает функции управления виртуальной машиной.
{: .prompt-tip }

Выберите в ниспадающем меню пункт Guest > Install/Upgrate VMware tools

Залогиньтесь в CentOS и подключите CDROM

```sh
$ sudo mkdir /mnt/cdrom  
$ sudo mount /dev/cdrom /mnt/cdrom
mount: block device /dev/cdrom is write-protected, mounting read-only
```

Перейдите в директорию `/mnt/cdroom`

```sh
$ cd /mnt/cdrom
```

Распакуйте VMware Tools в директорию `/tmp`

```sh
$ sudo tar -C /tmp -zxvf VMwareTools-5.5.3-34685.tar.gz
```

Запустите установку

```sh
$ cd /tmp/vmware-tools-distrib  
$ sudo ./vmware-install.pl --default
```

Чистим директорию `/tmp`

```sh
$ sudo rm -rf /tmp/vmware-tools-distrib
```
