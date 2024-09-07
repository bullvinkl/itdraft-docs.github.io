---
title: "Mikrotik WireGuard в качестве WireGuard клиента (peer)"
date: "2022-01-31"
categories: 
  - Security-System
  - Network-Equipment
tags: 
  - "mikrotik"
  - "routeos"
  - "vpn"
  - "wireguard"
image:
  path: /commons/open-source-proprietary.png
  alt: "Mikrotik WireGuard"
---

> В **RouterOS 7** появился **WireGuard**, который позиционирует себя как максимально простой в настройке VPN
{: .prompt-tip }

## Исходные данные

- WireGuard Server в сети предприятия: 192.16.1.0/24, UDP-порт опубликован
- Туннельная сеть: 172.16.30.0/24
- Туннельный IP WireGuard Server: 172.16.30.1
- Туннельный IP WireGuard Peer: 172.16.30.9
- Домашняя сеть: 192.168.11.0/24
- Домашний роутер (Mikrotik, RouteOS 7): 192.168.11.1

## Настройка WireGuard клиента

Запускаем Winbox, переходим пункт меню `wireguard` создаем новый интерфейс

```
Name: wireguard1
MTU: 1420
```

![](/assets/img/posts/2022/01/31/wg_int.png){: w="300" }

После сохранения должны сгенерироваться приватный и публичный ключи

Далее идем: `IP -> Addresses`, добавляем туннельный ip-адрес

```
Address: 172.16.30.9/24
Network: 172.16.30.0
Interface: wireguard1 (из прошлого шага)
```

![](/assets/img/posts/2022/01/31/wg_addr.png){: w="300" }

Возвращаемся в пункт меню Wireguard, вкладка `Peers`. Будем настраивать наш роутер как wireguard peer

```
Interface: wireguard1
Publiv Key: вставляем публичный ключ wireguard сервера
Endpoint: публичный ip wireguard сервера
Endpoint Port: UDP-порт wireguard сервера
Allowed Address: 172.16.30.1/32 - туннельный ip wireguard сервера
   192.168.1.0/24 - сеть предприятия
```

![](/assets/img/posts/2022/01/31/wg_peer.png){: w="300" }

Добавляем маршрутизацию, переходим: `IP -> Routes`

Тут сам должен был появиться маршрут:

```
Dst Address: 172.16.30.0/24
Gateway: wireguard1
Distance: 0
```

Добавляем маршрут до сети предприятия

```
Dst Address: 192.168.1.0/24
Gateway: wireguard1
```

![](/assets/img/posts/2022/01/31/wg_routes.png){: w="300" }

## Настройка WireGuard сервера

Редактируем конфиг сервера, добавляем пира

```sh
$ sudo nano /etc/wireguard/wg0.conf
...
[Peer]
# Client 3
PublicKey = %публичный ключ с клиента (с нашего роутера)%
AllowedIPs = 172.16.30.9/32, 192.168.11.0/24
```

`AllowedIPs` - список разрешенных ip, в том числе наша домашняя сетка (иначе из нашей сети не будет доступа к удаленной сети)

Перезапускаем wireguard server

```sh
$ sudo systemctl restart wg-quick@wg0
```

Теперь из домашней сети должны быть доступны ПК из сети предприятия
