---
title: "Настраиваем IPv6 от Ростелеком в роутере Mikrotik"
date: "2021-09-02"
categories: 
  - Network-Service
  - Network-Equipment
tags: 
  - "ipv6"
  - "mikrotik"
  - "rostelecom"
coverImage: "working-on-computer.jpg"
image:
  path: /commons/working-on-computer.jpg
  alt: "IPv6 от Ростелеком в роутере Mikrotik"
---

> **IPv6** — новая версия интернет-протокола, призванная решить проблемы, с которыми столкнулась предыдущая версия при её использовании в Интернете, за счёт целого ряда принципиальных изменений. Протокол был разработан IETF. Длина адреса IPv6 составляет 128 бит, в отличие от адреса IPv4, длина которого равна 32 битам.
{: .prompt-tip }

Вначале активируем пакет ipv6 в Mikrotik и перезагружаемся

```sh
System - Packages
System - Reboot
```

![](/assets/img/posts/2021/09/02/mik01.png)

Добавляем запись в DHCPv6

```
IPv6 - DHCP Client
```

Заполняем поля

```
Interface - ether1 (к которому подключен кабель от провайдера)
Request: address, perfix
Pool Name: любое имя, например ipv6-pool
Pool Prefix Lenght: 64
Use Peer DNS - отмечаем галкой (использовать DNS провайдера)
Add Default Route - отмечаем галкой
```

![](/assets/img/posts/2021/09/02/mik02.png){: w="300" }

Проверяем, для этого заходим во вкладку Status

![](/assets/img/posts/2021/09/02/mon03.png){: w="300" }

Включаем раздачу IPv6 в локалку

```
IPv6 - Addresses
```

Заполняем поля

```
Address: ::/64 - после того, как заполним остальные поля и нажмем ОК строчка поменяется
From Pool: ipv6-pool - то что указывали в прдыдущем этапе
Interface: bridge - наши локальные lan-порты
EUI64 - отмечаем галкой
Advertise - отмечаем галкой
```

![](/assets/img/posts/2021/09/02/mon04.png){: w="300" }

Меняем запись в сервисе Neighbor Discovery

```
IPv6 - ND
```

Изменяем текущую запись, или деактивируем ее и создаем новую

```
Interface: bridge
RA Interval: 20-60
RA Delay: 3
RA lifetime: 300
Hop Limit: 64
Advertise MAC Address - отмечаем галкой
Advertise DNS - отмечаем галкой
Other Configuration - отмечаем галкой
```

![](/assets/img/posts/2021/09/02/mon05.png){: w="300" }

Добавляем запись в DHCPv6 Server

```
IPv6 - DHCP Server
```

Заполняем поля

```
Name: любое имя, например my-dhcp
Interface: bridge
Address Pool6: ipv6-pool - то что добавляли в одном из первых шагов
Lease Time: 3d 00:00:00
Allow Dual Stack Queue - отмечаем галкой
Route Distance: 1
```

![](/assets/img/posts/2021/09/02/mon06.png){: w="300" }

Настраиваем базовые правила Firewall для IPv6

```
IPv6 - Firewall
```

![](/assets/img/posts/2021/09/02/mon07.png){: w="300" }

Быстрей это будет сделать через терминал

```sh
/ipv6 firewall filter
add chain=input action=drop connection-state=invalid 
add chain=input action=accept connection-state=established,related in-interface=ether1
add chain=forward action=accept connection-state=established,related in-interface=ether1 out-interface=bridge 
add chain=input action=accept protocol=icmpv6 
add chain=forward action=accept protocol=icmpv6
add chain=input action=accept protocol=udp in-interface=ether1 dst-port=546 
add chain=forward action=accept in-interface=bridge out-interface=ether1
add chain=input action=drop
```

где

```
2 строка: блокировать все «неправильные» соединения
3-4 строки: пропускать уже установленные соединения
5-6 строки: пропускать ICMP-пакеты, но ограничить лимит
7 строка: разрешить соединения от вашего провайдера по протоколу UDP на порт 546 - без этого правила вы не получите адрес IPv6 по DHCPv6 от провайдера
8 строка: пропускать все из вашей локальной сети в Интернет
9-10 строки: заблокировать все остальное
```

Настраиваем раздачу DNS роутером

```
IP - DNS
```

Заполняем поля (используем DNS от CloudeFlare)

```
Servers: 2606:4700:4700::1111
         2606:4700:4700::1001
         1.1.1.1
         1.0.0.1
Dynamic Servers: выдал провайдер
Use DoH Server: https://1.1.1.1/dns-query - включаем DNS over HTTPS
Verify DoH Certificate - отмечаем галкой
Allow Remote Requests - отмечаем галкой
Max USP Packet Size: 4096
Query Server Timeoute: 2.000
Query Total Timeoute: 10.000
Max. Concurrent Queries: 100
Max. Concurrent Sessions: 20
Cache Size 2048
Cache Max TTL: 7d 00:00:00
```

![](/assets/img/posts/2021/09/02/mon08.png){: w="300" }

Проверяем. Запускаем командную строку и выполняем

```
$ ping -6 ya.ru
```

![](/assets/img/posts/2021/09/02/mon09.png){: w="300" }

Запускаем браузер и переходим на тестовую ipv6 страницу google

```
http://ipv6test.google.com
```
