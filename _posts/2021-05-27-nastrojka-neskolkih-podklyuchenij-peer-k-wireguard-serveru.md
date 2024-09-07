---
title: "Настройка нескольких подключений (peer) к WireGuard серверу"
date: "2021-05-27"
categories: 
  - Security-System
tags: 
  - "centos"
  - "debian"
  - "linux"
  - "peer"
  - "ubuntu"
  - "wireguard"
image:
  path: /commons/best-free-proxies.png
  alt: "Настройка нескольких подключений к Wireguard"
---

> **WireGuard** - это протокол связи и бесплатное программное обеспечение с открытым исходным кодом, реализующее зашифрованные виртуальные частные сети, и был разработан с учетом простоты использования, высокой производительности и низкой поверхности для атак.
{: .prompt-tip }

Устанавливаем Wireguard на каждую машину ([рассматривали в одной из предыдущих статей]({% post_url 2020-10-29-ustanovka-vpn-servera-wireguard-v-centos-8-site-to-site-vpn %}))

На каждой машине необходимо сгенерить пару ключей (от пользователя root): публичный и приватный

```sh
$ sudo su
# cd /etc/wireguard
# wg genkey | tee privatekey | wg pubkey > publickey
```

## Настраиваем конфиг машины, которая будет в роли Wireguard сервера

Редактируем конфигурационный файл wg0.conf

```sh
$ sudo nano /etc/wireguard/wg0.conf

[Interface]
Address = 172.16.0.1/24
SaveConfig = false
ListenPort = 51820
PrivateKey = %Private key Server%
# Правила для iptables и маршрутизации
PostUp = iptables -I FORWARD -i %i -j ACCEPT
PostUp = iptables -I FORWARD -o %i -j ACCEPT
PostUp = ip route add 192.168.1.0/24 via 172.16.0.2 dev %i
PostUp = ip route add 192.168.2.0/24 via 172.16.0.3 dev %i
PostDown = iptables -D FORWARD -i %i -j ACCEPT
PostDown = iptables -D FORWARD -o %i -j ACCEPT
PostDown = ip route delete 192.168.1.0/24 via 172.16.0.2 dev %i
PostDown = ip route delete 192.168.2.0/24 via 172.16.0.3 dev %i
Table = off

[Peer]
# Client 1
PublicKey = %Publick key Client 1%
AllowedIPs = 172.16.0.2/32,192.168.1.0/24
#AllowedIPs = 0.0.0.0/0

[Peer]
# Client 2
PublicKey = %Publick key Client 2%
AllowedIPs = 172.16.0.3/32,192.168.2.0/24
```

> В случае нескольких пиров, параметр AllowedIPs должен содержать ip-адрес wireguard-клиента (пира) с маской 32, и разрешенные сети (192.168.х.0/24) не должны пересекаться. Иначе не заработает.

## Настраиваем конфиг для Client 1

Редактируем конфигурационный файл wg0.conf

```sh
$ sudo nano /etc/wireguard/wg0.conf

[Interface]
Address = 172.16.0.2/24
SaveConfig = false
PrivateKey = %Private key Client 1%
# Правила для iptables и маршрутизации
PostUp = iptables -I FORWARD -i %i -j ACCEPT;
PostUp = iptables -I FORWARD -o %i -j ACCEPT;
PostUp = ip route add 192.168.1.0/24 via 172.16.0.1 dev %i;
PostDown = iptables -D FORWARD -i %i -j ACCEPT;
PostDown = iptables -D FORWARD -o %i -j ACCEPT;
PostDown = ip route delete 192.168.1.0/24 via 172.16.0.1 dev %i;
Table = off

[Peer]
PublicKey = %Public key Server%
AllowedIPs = 0.0.0.0/0
Endpoint = %ip_server%:51820
PersistentKeepalive = 25
```

## Настраиваем конфиг для Client 2

Редактируем конфигурационный файл wg0.conf

```sh
$ sudo nano /etc/wireguard/wg0.conf

[Interface]
Address = 172.16.0.3/24
SaveConfig = false
PrivateKey = %Private key Client 3%
# Правила для iptables и маршрутизации
PostUp = iptables -I FORWARD -i %i -j ACCEPT;
PostUp = iptables -I FORWARD -o %i -j ACCEPT;
PostUp = ip route add 192.168.2.0/24 via 172.16.0.1 dev %i;
PostDown = iptables -D FORWARD -i %i -j ACCEPT;
PostDown = iptables -D FORWARD -o %i -j ACCEPT;
PostDown = ip route delete 192.168.2.0/24 via 172.16.0.1 dev %i;
Table = off

[Peer]
PublicKey = %Public key Server%
AllowedIPs = 0.0.0.0/0
Endpoint = %ip_server%:51820
PersistentKeepalive = 25
```

## Проверка подключения клиентов к Wireguard серверу

Для начала поднимаем интерфейс wg0 вначале на Wireguard сервере, а затем на всех клиентах

```sh
$ sudo wg-quick up wg0
```

Затем смотрим коннекты на Wireguard сервере

```
interface: wg0
  public key: %Public key Server%
  private key: (hidden)
  listening port: 51820

peer: %Publick key Client 1%
  endpoint: %ip-client1:port%
  allowed ips: 172.16.0.2/32, 192.168.1.0/24
  latest handshake: 1 minute, 12 seconds ago
  transfer: 156.40 MiB received, 387.80 MiB sent

peer: %Publick key Client 2%
  endpoint: %ip-client2:port%
  allowed ips: 172.16.0.3/32, 192.168.2.0/24
  latest handshake: 1 minute, 52 seconds ago
  transfer: 6.10 KiB received, 1.87 KiB sent
```

Если в пире присутствует строка `endpoint` с ip-адресом, значит соединение клиента с сервером установлено.
