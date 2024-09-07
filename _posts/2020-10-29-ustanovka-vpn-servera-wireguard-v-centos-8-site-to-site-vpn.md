---
title: "Установка VPN сервера Wireguard в Centos 8. Site-to-site VPN"
date: "2020-10-29"
categories: 
  - Security-System
tags: 
  - "centos"
  - "vpn"
  - "wireguard"
  - "firewall"
image:
  path: /commons/password-strength-in-2019-two-factor-authentication.png
  alt: "Установка VPN сервера Wireguard"
---

> **WireGuard** - это бесплатное программное приложение с открытым исходным кодом и протокол связи, который реализует методы виртуальной частной сети для создания безопасных соединений «точка-точка» в маршрутизируемых или мостовых конфигурациях.
{: .prompt-tip }

## Настройка главного сервера

Добавляем репозитории EPEL и Elrepo

```sh
$ sudo dnf -y install epel-release elrepo-release
```

Проверяем, подключен ли нужный драйвер

```sh
$ sudo lsmod | grep 8021q
8021q                  40960  0
garp                   16384  1 8021q
mrp                    20480  1 8021q
```

Если этих строк нет, добавляем драйвер в ядро

```sh
$ sudo modprobe 8021q
```

Устанавливаем wireguard

```sh
$ sudo dnf makecache
$ sudo dnf install -y kmod-wireguard wireguard-tools
```

Генерируем приватный и публичный ключи

```sh
$ sudo su
# cd /etc/wireguard
# wg genkey | tee privatekey | wg pubkey > publickey
```

Настраиваем конфигурационный файл для интерфейса wg0 и меняем права доступа

```sh
$ sudo touch /etc/wireguard/wg0.conf
$ sudo chmod 600 /etc/wireguard/{privatekey,wg0.conf}
```

Включаем Forwarding

```sh
$ echo "net.ipv4.ip_forward=1" | sudo tee -a /etc/sysctl.d/99-sysctl.conf
$ echo "net.ipv4.conf.all.forwarding=1" | sudo tee -a /etc/sysctl.d/99-sysctl.conf
$ echo "net.ipv6.conf.all.forwarding=1" | sudo tee -a /etc/sysctl.d/99-sysctl.conf
$ sysctl -p
```

Открываем порт (будем использовать порт 41321)

```sh
$ sudo firewall-cmd --permanent --zone=public --add-port=41321/udp
$ sudo firewall-cmd --reload
```

Смотрим приватный/публичный ключи главного сервера

```sh
$ sudo cat /etc/wireguard/privatekey
SMWUid073s000000000tvGgeN/Ow4BX9gsLqXFY=
$ sudo cat /etc/wireguard/publickey
RAalpQDMW000000000M761Vc56ugguVupB8ig=
```

Настраиваем интерфейс wg0

```sh
$ sudo nano /etc/wireguard/wg0.conf
[Interface]
Address = 172.16.30.1/24
# Отключаем перезапись конфига клиентом
SaveConfig = false
ListenPort = 41321
# Private Key of first Server
PrivateKey = SMWUid073scI+SyL/000000000000/Ow4BX9gsLqXFY=
PostUp = firewall-cmd --zone=public --add-port 41321/udp
PostUp = firewall-cmd --zone=public --add-masquerade
PostDown = firewall-cmd --zone=public --remove-port 41321/udp
PostDown = firewall-cmd --zone=public --remove-masquerade

[Peer]
# Public Key of second Server
PublicKey = vQyGpGrxCog0000h87mFHLt3Nkb4aJcTieq/yIJJ3Y=
AllowedIPs = 172.16.30.2/32
```

Команды для управления wireguard

```sh
$ sudo wg-quick up wg0
$ sudo wg-quick down wg0
$ sudo wg show wg0
interface: wg0
  public key: RAalpQD0000Z7fP5L000000000006ugguVupB8ig=
  private key: (hidden)
  listening port: 41321
```

Добавляем сервис в автозагрузку и запускаем его

```sh
$ sudo systemctl enable --now wg-quick@wg0
```

Проверяем, появился ли интерфейс, и доступен ли порт

```sh
$ ip a
$ ss -nltup
```

## Настройка второстепенного сервера

Установка производится аналогичным образом

Смотрим приватный/публичный ключи

```sh
$ sudo cat /etc/wireguard/privatekey 
SNtUeP3cn700000000Zc0000pDi4MC1nl8AoR28=
$ sudo cat /etc/wireguard/publickey 
vQyGpGrx00000000000000000cTieq/yIJJ3Y=
```

Настраиваем интерфейс wg0

```sh
$ sudo nano /etc/wireguard/wg0.conf
[Interface]
Address = 172.16.30.2/24
SaveConfig = true
ListenPort = 41321
# Private Key of second Server
PrivateKey = SNtUeP3cn7Y26zy0000Z00000000Di4MC1nl8AoR28=

[Peer]
# Public Key of first Server
PublicKey = RAalpQDMWkMZ7f0000sz00000000000gguVupB8ig=
AllowedIPs = 172.16.30.1/32
# public ip main server
Endpoint = 8.8.8.8:41321
```

Добавляем сервис в автозагрузку и запускаем его

```sh
$ sudo systemctl enable --now wg-quick@wg0
```

Либо можно воспользоваться командами управления wireguard

```sh
$ sudo wg-quick up wg0
$ sudo wg-quick down wg0
$ sudo wg show wg0
```
