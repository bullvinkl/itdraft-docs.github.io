---
title: "Device eth0 does not seem to be present, delaying initialization"
date: "2017-03-01"
categories: 
  - Linux
tags: 
  - "centos"
image:
  path: /commons/1c84bc2f-5343-4bee-b45c-60299c453816.png
  alt: "Device eth0 does not seem to be present, delaying initialization"
---

> Данная ошибка гласит о том, что в системе отсутствует данная карта, а это значит что в папке /etc/sysconfig/network-scripts есть файл настроек для сетевой карты ifcfg-eth0, но самой карты нету в device файле. Её нету и в настройках девайсов в файле /etc/udev/rules.d/70-persistant-net.rules.
{: .prompt-tip }

Проверяем сеть:

```sh
$ sudo ifconfig
lo Link encap:Local Loopback
inet addr:127.0.0.1 Mask:255.0.0.0
inet6 addr: ::1/128 Scope:Host
UP LOOPBACK RUNNING MTU:16436 Metric:1
RX packets:0 errors:0 dropped:0 overruns:0 frame:0
TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
collisions:0 txqueuelen:0
RX bytes:0 (0.0 b) TX bytes:0 (0.0 b)
```

Пробуем запустить сетевой интерфейс Eth0

```sh
$ sudo ifup eth0
```

Валится ошибка: `Device eth0 does not seem to be present, delaying initialisation`

Решение:  
Редактируем файл `/etc/udev/rules.d/70-persistent-net.rules` и прописываем верный MAC `08:00:27:fe:c1:03`

```
$ sudo nano /etc/udev/rules.d/70-persistent-net.rules
...
# PCI device 0x8086:0x100e (e1000)
SUBSYSTEM=="net", ACTION=="add", DRIVERS=="?*", ATTR{address}=="08:00:27:fe:c1:03", ATTR{type}=="1", KERNEL=="eth*", NAME="eth0"
...
```

Редактируем файл `/etc/sysconfig/network-scripts/ifcfg-eth0`, где так же прописываем верный MAC

```
$ sudo nano /etc/sysconfig/network-scripts/ifcfg-eth0
```

Перезагружаем систему

```
$ sudo reboot
```

Проверяем

```
$ sudo ifconfig
eth0 Link encap:Ethernet HWaddr 08:00:27:FE:C1:03
inet addr:192.168.1.99 Bcast:xxxxxxxx Mask:255.255.255.0
inet6 addr: fe80::a00:27ff:fefe:c103/64 Scope:Link
UP BROADCAST RUNNING MULTICAST MTU:1500 Metric:1
RX packets:4400 errors:0 dropped:0 overruns:0 frame:0
TX packets:129 errors:0 dropped:0 overruns:0 carrier:0
collisions:0 txqueuelen:1000
RX bytes:387597 (378.5 KiB) TX bytes:19567 (19.1 KiB)
```
