---
title: "Настройка статического IP в Debian 10"
date: "2019-08-26"
categories: 
  - Linux
tags: 
  - "debian"
  - "network"
image:
  path: /commons/programming-flat-60074276.jpg
  alt: "Настройка статического IP"
---

> **Debian** — операционная система, состоящая из свободного ПО с открытым исходным кодом. В настоящее время Debian GNU/Linux — один из самых популярных и важных дистрибутивов GNU/Linux, в первичной форме оказавший значительное влияние на развитие этого типа ОС в целом. Debian может использоваться в качестве операционной системы как для серверов, так и для рабочих станций.
{: .prompt-tip }

Смотрим какие сетевые интерфейсы подняты в системе

```sh
$ ip addr show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:33:07:93 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.15/24 brd 192.168.1.255 scope global dynamic enp0s3
       valid_lft 86277sec preferred_lft 86277sec
    inet6 fdd4:a148:ea1:7a00:a00:27ff:fe33:793/64 scope global dynamic mngtmpaddr 
       valid_lft 7078sec preferred_lft 3478sec
    inet6 fe80::a00:27ff:fe33:793/64 scope link 
       valid_lft forever preferred_lft forever
```

Редактируем файл настроек сети

```sh
$ sudo nano /etc/network/interfaces
```

Меняем параметр `dhcp` на `static`, и дописываем параметры сети

```
iface enp0s3 inet static
	address 192.168.1.15
	netmask 255.255.255.0
	network 192.168.1.0
	broadcast 192.168.255.255
	gateway 192.168.1.1
	dns-nameservers 192.168.1.1
	#dns-nameservers 8.8.8.8 8.8.4.4
```

Перезапускаем сеть

```sh
$ systemctl restart networking
```

Проверяем

```sh
$ ip addr show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 92:00:b4:b2:f5:ac brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.15/24 brd 192.168.12.255 scope global enp0s3
       valid_lft forever preferred_lft forever
    inet6 fe80::9000:b4ff:feb2:f5ac/64 scope link
       valid_lft forever preferred_lft forever
```

Если сетевой интерфейс не поднялся, выполняем команду

```sh
$ sudo ifup enp0s3
```
