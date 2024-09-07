---
title: "Установка и настройка PPTP VPN в Centos 7"
date: "2018-04-24"
categories: 
  - Security-System
tags: 
  - "centos"
  - "pptp"
  - "vpn"
image:
  path: /commons/1292838_f55d.jpg
  alt: "Установка и настройка PPTP VPN в Centos"
---

> **PPTP** (Point-to-Point Tunneling Protocol) - это устаревший туннельный протокол, разработанный компанией Microsoft. Он позволяет компьютеру устанавливать защищенное соединение с сервером, создавая специальный туннель в стандартной, незащищенной сети.
{: .prompt-tip }

Добавляем репозиторий EPEL

```sh
$ sudo yum install epel-release
```

Обновляемся

```sh
$ sudo yum update && yum upgrade
```

Ставим софт

```sh
$ sudo yum install ppp pptpd nano
```

Редактируем файл конфигурации, добавив в самом низу

```sh
$ sudo nano /etc/pptpd.conf
localip 10.10.0.1
remoteip 10.10.0.100-199
```

Редактируем файл с настройками, добавив в самом низу

```sh
$ sudo nano /etc/ppp/options.pptpd
ms-dns 8.8.8.8
ms-dns 8.8.4.4
```

Добавляем пользователей

```sh
$ sudo nano /etc/ppp/chap-secrets
# Secrets for authentication using CHAP
# client server secret IP addresses
username pptpd password *
```

Включаем forward

```sh
$ sudo nano /etc/sysctl.conf
...
net.ipv4.ip_forward = 1
```

Применяем изменение

```sh
$ sudo sysctl -p
```

Редактируем файл, без этого некоторые сайты не грузятся

```sh
$ sudo nano /etc/ppp/ip-up
/sbin/ifconfig $1 mtu 1400
```

Добавляем правила firewall

```sh
$ sudo nano /etc/firewalld/services/pptp.xml
```

```sh
$ sudo firewall-cmd --permanent --new-service=pptp
$ sudo firewall-cmd --permanent --zone=public --add-service=pptp
$ sudo firewall-cmd --permanent --zone=public --add-masquerade
$ sudo firewall-cmd --reload
```

Запускаем службу и добавляем в автозагрузку

```sh
$ sudo systemctl start pptpd
$ sudo systemctl enable pptpd.service
```
