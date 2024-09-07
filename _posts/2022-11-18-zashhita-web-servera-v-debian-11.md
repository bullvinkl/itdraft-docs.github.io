---
title: "Защита Web-сервера с помощью UFW, WireGuard и Dnsmasq в Debian 11"
date: "2022-11-18"
categories: 
  - Linux
  - UFW
  - WireGuard
tags: 
  - "debian"
  - "dnsmasq"
  - "docker"
  - "ufw"
  - "ufw-docker"
  - "wireguard"
image:
  path: /commons/virtualisationimage.png
  alt: "Защита Web-сервера"
---

> **UFW (Uncomplicated Firewall)** — это утилита для конфигурирования межсетевого экрана Netfilter. Она использует интерфейс командной строки, состоящий из небольшого числа простых команд.
{: .prompt-tip }

В статье рассмотрен один из вариантов защиты Web севера, который показался мне интересным

Принцип действия следующий:

- На сервере открываем входящие web и vpn порты

- Подключение по SSH разрешено при активной VPN сессии

- Подключение к админке сайта разрешено при активной VPN сессии

## Установка и настройка WireGuard

Устанавливаем дистрибутив и генерим пару ключей

```sh
$ sudo apt install wireguard -y
$ sudo su
$ cd /etc/wireguard/
$ umask 077; wg genkey | tee privatekey | wg pubkey > publickey
$ exit
```

Редактируем конфиг wireguard

```sh
$ sudo nano /etc/wireguard/wg0.conf
[Interface]
PrivateKey = # Приватный ключ, который сгенерировали выше
Address = 172.16.22.1/24
ListenPort = 41821

[Peer]
PublicKey = # Публичный ключ клиента №1
AllowedIPs = 172.16.22.2/32

[Peer]
PublicKey = # Публичный ключ клиента №2
AllowedIPs = 172.16.22.3/32
```

Запускаем сервис и добавляем его в автозагрузку

```sh
$ sudo systemctl start wg-quick@wg0
$ sudo systemctl enable wg-quick@wg0
```

## Настройка WireGuard на клиенте

Приводим конфиг wireguard к виду

```sh
[Interface]
PrivateKey = # Приватный ключ клиента, генерируется сам
Address = 172.16.22.2/32
DNS = 172.16.22.1

[Peer]
PublicKey = # Публичный ключ сервера
AllowedIPs = 172.16.22.1/32
Endpoint = itdraft.ru:41821
PersistentKeepalive = 25
```

## Установка и настройка DNS-сервера Dnsmasq

Устанавливаем дистрибутив и редактируем конфиг

```sh
$ sudo apt -y install dnsmasq
$ sudo nano /etc/dnsmasq.d/dnsmasq.conf

# Слушаем интерфейс wireguard
listen-address=172.16.33.1
bind-interfaces

# не передавать значения из /etc/hosts
no-hosts

# подмена ip
address=/itdraft.ru/172.16.22.1
```

Перезапускаем сервис

```sh
$ sudo systemctl restart dnsmasq
```

Таким образом, при подключении по VPN, локальный DNS-сервер подменяет белый ip на серый.  
Благодаря этому мы можем защитить админку сайта соответствующими настройками Nginx: разрешить доступ к директории только из сетки Wireguard

Осталось защитить SSH-порт

## Установка и настройка UFW

Устанавливаем дистрибутив

```sh
$ sudo apt -y install ufw
```

Проверяем состояние UFW (по-умолчанию UFW не активен)

```sh
$ sudo ufw status verbose
Status: inactive
```

Задаем значения по умолчанию: запрещаем входящие соединения и разрешаем исходящие

```sh
$ sudo ufw default deny incoming
$ sudo ufw default allow outgoing
```

Добавляем правило: разрешаем подключение к ssh только из сети wireguard

```sh
$ sudo ufw allow proto tcp from 172.16.22.0/24 to any port 22
```

Открываем остальные порты

```sh
$ sudo ufw allow http
$ sudo ufw allow https
$ sudo ufw allow 41821/udp
```

Активируем UFW

```sh
$ sudo ufw enable
```

Другие команды UFW

Отключаем UFW

```sh
$ sudo ufw disable
```

Смотрим правила и удаление правило 2

```sh
$ sudo ufw status numbered
$ sudo ufw delete 2
```

Сбросить настройки

```sh
$ sudo ufw reset
```

## Настройка связки UFW + Docker

Скачиваем утилиту ufw-docker, делаем её исполняемой и устанавливаем

```sh
$ sudo wget -O /usr/local/bin/ufw-docker https://github.com/chaifeng/ufw-docker/raw/master/ufw-docker
$ sudo chmod +x /usr/local/bin/ufw-docker
$ sudo ufw-docker install
```

Перезапускаем UFW

```sh
$ sudo systemctl restart ufw
```

Открываем web-порты

```sh
$ sudo ufw-docker allow 80
$ sudo ufw-docker allow 443
```

Но у меня заработала связка UFW + Docker после следующих действий:

Смотрим наши сети

```sh
$ docker network ls
NETWORK ID     NAME           DRIVER    SCOPE
639665111b6c   proj_default   bridge    local
5ff37bbe516b   bridge         bridge    local
421941e27957   host           host      local
8cf544723e76   none           null      local
```

Смотрим выделенный диапазон ip сети `proj_default`

```sh
$ docker inspect proj_default
...
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.21.0.0/16",
                    "Gateway": "172.21.0.1"
                }

...
```

Разрешаем входящий трафик для этой сети

```sh
$ sudo ufw allow in from 172.21.0.0/16
```
