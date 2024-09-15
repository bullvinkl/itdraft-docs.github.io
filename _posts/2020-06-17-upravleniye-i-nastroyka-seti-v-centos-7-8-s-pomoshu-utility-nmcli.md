---
title: "Управление и настройка сети в Centos 7/8 с помощью утилиты nmcli"
date: "2020-06-17"
categories: 
  - Network-Service
tags: 
  - "centos"
  - "nmcli"
image:
  path: /commons/1162000_fab6_2.jpg
  alt: "Управление и настройка сети с помощью nmcli"
---

> **nmcli** (network manager command-line interface) - утилита для настройки сети, которая позволяет использовать Network Manager в консоли
{: .prompt-tip }

Запустим Network Manager, проверяем статус

```sh
$ sudo systemctl start NetworkManager
$ sudo systemctl status NetworkManager
```

## Информация об интерфейсах

Посмотреть соединения

```sh
$ nmcli connection show
или
$ nmcli con show
или
$ nmcli c s

NAME           UUID                                  TYPE      DEVICE 
System ens192  085a58e2-18f3-4a76-8717-d1f53aa67642  ethernet  ens192
```

Посмотреть только активные соединения

```sh
$ nmcli con show -a
```

Посмотреть полую информацию обо всех интерфейсах

```sh
$ nmcli dev show
```

Посмотреть полую информацию об интерфейсе `ens192`

```sh
$ nmcli dev show ens192
```

Посмотреть статус интерфейсов (активные/не активные)

```sh
$ nmcli dev status
DEVICE  TYPE      STATE      CONNECTION    
ens192  ethernet  connected  System ens192 
lo      loopback  unmanaged  -- 
```

## Настройка интерфейсов

Поднять или отключить интерфейс

```sh
$ nmcli con down <connectionName>
$ nmcli con up <connectionName>

Пример:
$ nmcli con down "System ens192"
$ nmcli con up "System ens192"
```

Изменить IP-адрес

```sh
$ sudo nmcli con mod "System ens192" ipv4.addresses 192.168.1.89/24
```

Изменить шлюз (gateway)

```sh
$ sudo nmcli con mod "System ens192" ipv4.gateway 192.168.1.1
```

Изменить DNS

```sh
$ sudo nmcli con mod "System ens192" ipv4.dns 8.8.8.8,8.8.4.4
```

Добавить DNS сервер

```sh
$ sudo nmcli con mod "System ens192" +ipv4.dns 1.1.1.1
```

Удалить DNS сервер

```sh
$ sudo nmcli con mod "System ens192" -ipv4.dns 1.1.1.1
```

Либо изменить сетевые настройки (ip, gate, dns) одно командой

```sh
$ sudo nmcli con mod "System ens192" ipv4.addresses 192.168.1.89/24 ipv4.gateway 192.168.1.1 ipv4.dns 8.8.8.8,8.8.4.4
```

Задать `dns-search`

```sh
$ sudo nmcli con mod "System ens192" ipv4.dns-search "domain1.local,domain2.local,domain3.local"
```

Изменить имя интерфейса "Wired connection 1" на "ens224" (DEVICE=ens224)

```sh
$ sudo nmcli con mod "Wired connection 1" connection.interface-name "ens224"
```

Изменить id интерфейса "System ens192" на "ens192" (NAME=ens192)

```sh
$ sudo nmcli con mod "System ens192" connection.id ens192
```

Добавить интерфейс

```sh
$ sudo nmcli con add con-name "static-ens224" ifname ens224 type ethernet ip4 192.168.1.76/24 gw4 192.168.1.1
```

Запустить добавленный интерфейс

```sh
$ sudo nmcli con up "static-ens224" iface ens224
```

Удалить добавленный интерфейс

```sh
$ sudo nmcli con del "static-ens224"
```

Изменить DHCP на StaticIP

```sh
$ sudo nmcli con mod "System ens192" ipv4.method manual
```

Изменить StaticIP на DHCP

```sh
$ sudo nmcli con mod "System ens192" ipv4.method auto
```

Включить авто подключение к сети (`ONBOOT=yes`)

```sh
$ sudo nmcli con mod "System ens192" connection.autoconnect yes
```

Игнорировать информацию DNS-сервера с DHCP-сервера (`PEERDNS=no`)

```sh
$ sudo nmcli con mod "System ens192" ipv4.ignore-auto-dns true
```

Не использовать предоставленный шлюз в качестве шлюза по умолчанию (`DEFROUTE=yes`)

```sh
$ sudo nmcli con mod "System ens192" ipv4.never-default no
```

Узнать интерфейс

```sh
$ nmcli -f NAME -m multiline con show
$ nmcli -f NAME -m multiline con show | awk '{ print $2; }'
```

## Маршрутизация

Добавить статический роутинг

```sh
$ sudo nmcli con mod "System ens192" +ipv4.routes "10.0.0.0/8 10.33.22.11"
```

Удалить статический роутинг

```sh
$ sudo nmcli con mod "System ens192" -ipv4.routes "10.0.0.0/8 10.33.22.1"
```

После добавления маршрутов необходимо перезапустить службу NetworkManager

```sh
$ sudo systemctl restart NetworkManager
```
