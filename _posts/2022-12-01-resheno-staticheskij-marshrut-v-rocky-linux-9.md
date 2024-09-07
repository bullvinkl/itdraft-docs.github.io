---
title: "[Решено] Статический маршрут в Rocky Linux 9"
date: "2022-12-01"
categories: 
  - Linux
tags: 
  - "network"
  - "rocky-linux"
  - "route"
image:
  path: /commons/no-release-repo.jpg
  alt: "Статический маршрут в Rocky Linux 9"
---

> **Статический маршрут** (Static Route) - это запись маршрутизации, настроенная вручную, без применения протоколов маршрутизации (например, RIP, OSPF, EIGRP и т.д.). В статическом маршруте пользователь определяет путь, по которому пакеты должны быть направлены к определённому адресу назначения.
{: .prompt-tip }

Настройки сетевого интерфейса в RHEL-like дистрибутивах полностью переделаны и теперь расположены

```
/etc/NetworkManager/system-connections/{NAME}.nmconnection
```

Смотрим имя нашего сетевого интерфейса

```sh
$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ens192: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:51:46:0c:82:29 brd ff:ff:ff:ff:ff:ff
    altname enp11s0
    inet 10.2.81.17/24 brd 10.2.81.255 scope global noprefixroute ens192
       valid_lft forever preferred_lft forever
    inet6 fe80::250:56ff:fe0c:9249/64 scope link noprefixroute 
       valid_lft forever prefer
```

Что бы вручную не вбивать маршруты по отдельности через утилиту nmtui, редактируем конфигурационный файл сетевых настроек

```sh
$ sudo nano /etc/NetworkManager/system-connections/ens192.nmconnection
...
[ipv4]
address1=10.2.81.17/24,10.2.81.1
dns=195.208.4.1;195.208.5.1;
method=manual
route1=10.3.0.0/24,10.2.81.254
route2=10.4.0.0/24,10.2.81.254
route3=10.5.0.0/24,10.2.81.254
route4=10.6.0.0/24,10.2.81.254
...
```

Перезапускаем NetworkManager

```sh
$ sudo systemctl restart NetworkManager
```

Проверяем

```sh
$ ip r
default via 10.2.81.1 dev ens192 proto static metric 100
10.2.81.0/24 dev ens192 proto kernel scope link src 10.2.81.17 metric 100
10.3.0.0/24 via 10.2.81.254 dev ens192 proto static metric 100
10.4.0.0/24 via 10.2.81.254 dev ens192 proto static metric 100
10.5.0.0/24 via 10.2.81.254 dev ens192 proto static metric 100
10.6.0.0/24 via 10.2.81.254 dev ens192 proto static metric 100
```
