---
title: "Настройка IP в CentOS"
date: "2015-07-29"
categories: 
  - Manuals
tags: 
  - "centos"
  - "ip"
  - "linux"
  - "network"
image:
  path: /commons/tux30-h.jpg
  alt: "Настройка IP в CentOS"
---

> **IP-адрес** (от англ. Internet Protocol Address) - это уникальный адрес, идентифицирующий устройство в Интернете или локальной сети. Он представляет собой числовой идентификатор, который позволяет устройствам обмениваться данными в компьютерной сети, работающей по протоколу TCP/IP.
{: .prompt-tip }

Редактируем файл `/etc/sysconfig/network-scripts/ifcfg-eth0`

```sh
$ sudo nano /etc/sysconfig/network-scripts/ifcfg-eth0
DEVICE=eth0
BOOTPROTO=static
HWADDR=00:01:02:03:04:05
ONBOOT=yes
DHCP\_HOSTNAME=hostname.my.domain
IPADDR= Ваш.IP.адрес
NETMASK=255.255.255.0
GATEWAY= IP.адрес.шлюза
TYPE=Ethernet
```

Перезапускаем сервис

```sh
$ sudo service network restart
```
