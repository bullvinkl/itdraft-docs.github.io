---
title: "Determining if ip address is already in use for device ethX"
date: "2017-03-01"
categories: 
  - Linux
tags: 
  - "centos"
image:
  path: /commons/1.png
  alt: "Determining if ip address is already in use for device ethX"
---
> В Linux-системах, когда вы пытаетесь настроить сетевой интерфейс с определенным IP-адресом, система может выдавать сообщение о том, что адрес уже используется на этом интерфейсе. Это происходит из-за автоматической проверки занятости IP-адреса перед его назначением.
{: .prompt-tip }

Во время работы с сетевой подсистемой операционных систем Centos может возникнуть ошибка

> Determining if ip address is already in use for device ethX
{: .prompt-danger }

Например при перезагрузке сетевой подсистемы операционной системы Linux RHEL/Centos ошибка будет выглядеть следующим образом:

```sh
$ sudo /etc/init.d/network restart
Shutting down interface eth0: [ OK ]
Shutting down loopback interface: [ OK ]
Bringing up loopback interface: [ OK ]
Bringing up interface eth0: Determining if ip address 192.168.2.15 is already in use for device eth0… [ OK ]
```

Никаких ошибок в выводе команды `ifconfig` при этом не будет.

Для исправления возникающего сообщения, необходимо добавить описание параметра `ARPCHECK=no` в файл настроек каждого сетевого интерфейса сетевой подсистемы операционной системы на базе Linux RHEL/Centos.

После добавления соответствующего параметра, файл настроек одного из сетевых интерфейсов будет выглядеть следующим образом:

```sh
$ sudo nano /etc/sysconfig/network-scripts/ifcfg-eth0
DEVICE=”eth0″
BOOTPROTO=none
NM_CONTROLLED=”yes”
ONBOOT=yes
TYPE=”Ethernet”
UUID=”a4568461-be96-4b7a-9rr0-07a5bqwcd3259″
HWADDR=00:1C:39:1C:E4:44
IPADDR=192.168.2.15
PREFIX=24
NETMASK=255.255.255.0
GATEWAY=192.168.2.254
ARPCHECK=no
DNS1=8.8.8.8
DEFROUTE=yes
IPV4_FAILURE_FATAL=yes
IPV6INIT=no
NAME=”System eth0″
```

После добавления необходимо параметра, ошибка выводиться на экран консоли терминала более не будет.
