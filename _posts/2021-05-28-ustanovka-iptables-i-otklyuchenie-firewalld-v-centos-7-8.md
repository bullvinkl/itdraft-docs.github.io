---
title: "Установка iptables и отключение firewalld в Centos 7 / 8"
date: "2021-05-28"
categories: 
  - Security-System
tags: 
  - "centos"
  - "firewall"
  - "iptables"
image:
  path: /commons/working-on-computer.jpg
  alt: "Установка iptables"
---

> **Firewalld** - это брандмауэр, используемый по умолчанию в Centos 7 и Centos 8, а также для RHEL 7 и RHEL 8.
{: .prompt-tip }

Чтобы начать использовать `iptables` в Centos, необходимо отключить `firewalld`, что бы избежать конфликтов в правилах межсетевого экранирования.

Останавливаем firewalld и убираем его из автозагрузки

```sh
$ sudo systemctl stop firewalld
$ sudo systemctl disable firewalld
```

Маскируем службу firewalld, чтобы она не вызывалась каким-либо другим сервисом

```sh
$ sudo systemctl mask --now firewalld
```

Проверяем

```sh
$ sudo systemctl status firewalld
● firewalld.service
   Loaded: masked (Reason: Unit firewalld.service is masked.)
   Active: inactive (dead)
```

Устанавливаем iptables

```sh
$ sudo yum install iptables-services -y
```

Запускаем службы и добавляем их в автозагрузку

```sh
$ sudo systemctl start iptables
$ sudo systemctl start ip6tables
$ sudo systemctl enable iptables
$ sudo systemctl enable ip6tables
```

Проверяем

```sh
$ sudo systemctl status iptables
● iptables.service - IPv4 firewall with iptables
   Loaded: loaded (/usr/lib/systemd/system/iptables.service; enabled; vendor preset: disabled)
   Active: active (exited) since Fri 2021-05-28 10:16:50 MSK; 39min ago
 Main PID: 4423 (code=exited, status=0/SUCCESS)
    Tasks: 0 (limit: 23516)
   Memory: 0B
   CGroup: /system.slice/iptables.service
```

Смотрим текущие правила iptables

```sh
$ sudo iptables -nvL
```
