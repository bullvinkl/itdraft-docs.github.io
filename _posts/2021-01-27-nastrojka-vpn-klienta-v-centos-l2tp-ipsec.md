---
title: "Настройка VPN-клиента в Centos. L2TP + IPSec"
date: "2021-01-27"
categories: 
  - Linux
tags: 
  - "ipsec"
  - "l2tp"
  - "strongswan"
  - "vpn"
  - "xl2tpd"
image:
  path: /commons/cloud-3406627_1280.jpg
  alt: "L2TP + IPSec"
---

> **Layer 2 Tunneling Protocol (L2TP)** - это протокол туннелирования второго уровня. Он является расширением протокола туннелирования «точка-точка» (PPTP), используемого поставщиками интернет-услуг (ISP) для обеспечения работы виртуальной частной сети VPN через Интернет. L2TP объединяет лучшие функции двух других протоколов туннелирования: PPTP от Microsoft и L2F от Cisco Systems.
{: .prompt-tip }

Исходные данные:

```
$VPN_SERVER_IP - Внешний IP адрес VPN-сервера
$VPN_IPSEC_PSK - IPSec Pre-Shared Key
$VPN_USER - VPN Username
$VPN_PASSWORD - VPN_password
$VPN_SERVER_ID - Серый IP адрес VPN. Если не знаете, то его можно увидеть в логах подключения к VPN
```

Устанавливаем необходимый софт

```sh
$ sudo yum -y install strongswan xl2tpd
```

Настраиваем `IPSec`

```sh
$ sudo nano /etc/ipsec.conf
[...]
# basic configuration

config setup
  nat_traversal=yes
  # strictcrlpolicy=yes
  # uniqueids = no

# Add connections here.

# Sample VPN connections

conn %default
  ikelifetime=60m
  keylife=20m
  rekeymargin=3m
  keyingtries=1
  keyexchange=ikev1
  authby=secret
  ike=3des-sha1-modp1024!
  esp=aes128-sha1!

conn vpn01
  keyexchange=ikev1
  left=%defaultroute
  auto=add
  authby=secret
  type=transport
  leftprotoport=17/1701
  rightprotoport=17/1701
  right=$VPN_SERVER_IP
  rightid=$VPN_SERVER_ID
```

Прописываем IPSec Pre-Shared Key

```sh
$ sudo nano /etc/ipsec.secrets
: PSK "$VPN_IPSEC_PSK"
```

Меняем права на файл ipsec.secrets

```sh
$ sudo chmod 600 /etc/ipsec.secrets
```

Настраиваем `strongswan`

```sh
$ sudo mv /etc/strongswan/ipsec.conf /etc/strongswan/ipsec.conf.old 2>/dev/null
$ sudo mv /etc/strongswan/ipsec.secrets /etc/strongswan/ipsec.secrets.old 2>/dev/null
$ sudo ln -s /etc/ipsec.conf /etc/strongswan/ipsec.conf
$ sudo ln -s /etc/ipsec.secrets /etc/strongswan/ipsec.secrets
```

Настраиваем `xl2tpd`

```sh
$ sudo nano /etc/xl2tpd/xl2tpd.conf
[...добавить в конец основного конфига...]
[lac vpn01]
lns = $VPN_SERVER_IP
ppp debug = yes
pppoptfile = /etc/ppp/options.l2tpd.client
length bit = yes
```

```sh
$ sudo nano /etc/ppp/options.l2tpd.client
ipcp-accept-local
ipcp-accept-remote
refuse-eap
require-chap
noccp
noauth
mtu 1460
mru 1460
noipdefault
defaultroute
usepeerdns
connect-delay 5000
name $VPN_USER
password $VPN_PASSWORD
```

Меняем права на файл options.l2tpd.client

```sh
$ sudo chmod 600 /etc/ppp/options.l2tpd.client
```

Создаем каталог и файл

```sh
$ sudo mkdir -p /var/run/xl2tpd
$ sudo touch /var/run/xl2tpd/l2tp-control
```

Перезапускаем службы xl2tpd и strongswan

```sh
$ sudo systemctl restart xl2tpd
$ sudo systemctl restart strongswan
```

Проверяем статус

```sh
$ systemctl status xl2tpd
$ systemctl status strongswan
```

Подключаемся к VPN.  
Поднимаем IPSec

```sh
$ sudo strongswan up vpn01
```

Затем L2TP

```sh
$ echo "c vpn01" | sudo tee /var/run/xl2tpd/l2tp-control
```

Проверяем, должен появиться сетевой интерфейс ppp

```sh
$ ip a
```

Для отключения от VPN: вначале отключаем L2TP

```sh
$ echo "d vpn01" | sudo tee /var/run/xl2tpd/l2tp-control
```

Затем IPSec

```sh
$ sudo strongswan down vpn01
```
