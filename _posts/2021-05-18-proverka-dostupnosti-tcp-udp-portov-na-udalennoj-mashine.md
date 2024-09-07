---
title: "Проверка доступности TCP / UDP портов на удаленной машине"
date: "2021-05-18"
categories: 
  - Manuals
tags: 
  - "netcat"
  - "nmap"
image:
  path: /commons/computer3.jpg
  alt: "Проверка доступности TCP / UDP портов"
---

> **netcat** - утилита Unix, позволяющая устанавливать соединения TCP и UDP, принимать оттуда данные и передавать их. Несмотря на свою полезность и простоту, данная утилита не входит ни в какой стандарт. 
{: .prompt-tip }

Для проверки доступности TCP / UDP портов на удаленной машине будем использовать утилиту `Netcat`

Установка:

```sh
$ sudo yum install nc    # для CentOS

$ sudo apt update          # Debian / Ubuntu
$ sudo apt install netcat 
```

Под ОС Windows эта утилита так же [доступна](https://eternallybored.org/misc/netcat/)

Проверяем TCP-порт

```sh
$ nc -zv 10.10.4.4 22
srv-app.local [10.10.4.4] 22 (ssh) open
```

Проверяем UDP-порт

```sh
$ nc -uv 10.10.4.2 123
srv-dc01.local [10.10.4.2] 123 (ntp) open
```

## UPD 05.01.2022

Проверяем доступность TCP / UDP порта через утилиту `nmap`. 

Устанавливаем утилиту

```sh
$ sudo dnf -y install nmap
```

Проверяем 22 TCP порт (SSH)

```sh
$ sudo nmap -p22 79.308.191.187

Starting Nmap 6.40 ( http://nmap.org ) at 2022-01-05 19:40 MSK
Nmap scan report for 79.308.191.187
Host is up (0.0015s latency).
PORT   STATE SERVICE
22/tcp open  ssh

Nmap done: 1 IP address (1 host up) scanned in 0.53 seconds
```

Проверяем 53 UDP порт (NTP)

```sh
$ sudo nmap -sU -p U:123 94.247.111.10

Starting Nmap 6.40 ( http://nmap.org ) at 2022-01-05 19:37 MSK
Nmap scan report for ntp.truenetwork.ru (94.247.111.10)
Host is up (0.049s latency).
PORT    STATE SERVICE
123/udp open  ntp

Nmap done: 1 IP address (1 host up) scanned in 0.47 seconds
```
