---
title: "Установка и настройка DHCP сервера и клиента в Centos 7"
date: "2018-11-30"
categories: 
  - Linux
tags: 
  - "dhcp"
image:
  path: /commons/password-strength-in-2019-two-factor-authentication.png
  alt: "настройка DHCP"
---

> **DHCP** (англ. Dynamic Host Configuration Protocol — протокол динамической настройки узла) — сетевой протокол, позволяющий компьютерам автоматически получать IP-адрес и другие параметры, необходимые для работы в сети TCP/IP. Данный протокол работает по модели «клиент-сервер».

## Установка DHCP сервера

DHCP сервер доступен в официальном репозитории. Для установки выполним команду в терминале

```sh
$ sudo yum install dhcp
```

Далее необходимо прописать сетевой интерфейс, для которого DHCP сервер будет обслуживать запросы

```sh
$ sudo nano /etc/sysconfig/dhcpd
DHCPDARGS=”eth0”
```

## Настройка DHCP сервера

Основной файл конфигурации DHCP сервера расположен /etc/dhcp/dhcpd.conf  
Скопируем шаблон и откроем его для редактирования

```sh
$ sudo cp /usr/share/doc/dhcp-4.2.5/dhcpd.conf.example /etc/dhcp/dhcpd.conf	
$ sudo nano /etc/dhcp/dhcpd.conf
option domain-name "domain.local";
option domain-name-servers ns1.domain.local, ns2.domain.local;
default-lease-time 3600; 
max-lease-time 7200;
authoritative;
subnet 192.168.1.0 netmask 255.255.255.0 {
        option routers                  192.168.1.1;
        option subnet-mask              255.255.255.0;
        option domain-search            "domain.local";
        option domain-name-servers      192.168.1.1;
        range   192.168.10.10   192.168.10.100;
        range   192.168.10.110   192.168.10.200;
}
```

Добавим сервис в автозагрузку, запустим его и проверим статус

```sh
$ sudo systemctl enable dhcpd
$ sudo systemctl start dhcpd
$ systemctl status dhcpd
```

Добавим разрешающее правило для файерволла

```sh
$ sudo firewall-cmd --zone=public --permanent --add-service=dhcp
$ sudo firewall-cmd --reload 
```

## Настройка DHCP клиента

Редактируем файл конфигурации сетевого интерфейса

```sh
$ sudo nano /etc/sysconfig/network-scripts/ifcfg-eth0
DEVICE=eth0
BOOTPROTO=dhcp
TYPE=Ethernet
ONBOOT=yes
```

Перезапускаем сетевую службу

```sh
$ sudo systemctl restart network
```