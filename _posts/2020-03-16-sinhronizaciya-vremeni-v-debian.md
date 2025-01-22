---
title: "Синхронизация времени в Debian"
date: "2020-03-16"
categories: 
  - Manuals
tags: 
  - "debian"
  - "ntp"
  - "systemd-timesyncd"
  - "timesyncd"
image:
  path: /commons/1382848_75d8_2.jpg
  alt: "Синхронизация времени"
---

> **Systemd-timesyncd** - встроенная служба для синхронизации времени компьютера с ntp-серверами. Эта служба реализует упрощенный клиент SNTP. В отличие сложных реализаций NTP, systemd-timesyncd представляет только клиентскую часть.
{: .prompt-tip }

## Настройка timesyncd

Смотрим текущий статус синхронизации времени

```sh
$ timedatectl status
      Local time: Mon 2020-03-16 09:06:15 MSK
  Universal time: Mon 2020-03-16 06:06:15 UTC
        RTC time: Mon 2020-03-16 06:05:06
       Time zone: Europe/Moscow (MSK, +0300)
     NTP enabled: no
NTP synchronized: no
 RTC in local TZ: no
      DST active: n/a
```

Включаем использование `systemd-timesyncd` для синхронизации времени

```sh
$ sudo timedatectl set-ntp true
```

Настроим `systemd-timesyncd`.  
Конфигурационный файл расположен тут: `/etc/systemd/timesyncd.conf`

```sh
$ echo 'Servers=192.168.1.1 192.168.1.2' | sudo tee -a /etc/systemd/timesyncd.conf > /dev/null
```

где `192.168.1.1, 192.168.1.2` - ntp серверы

По-умолчанию служба выключена. Включаем и перезапускаем службу systemd-timesyncd

```sh
$ sudo systemctl enable systemd-timesyncd
$ sudo systemctl restart systemd-timesyncd
```

Проверяем статус

```sh
$ systemctl status systemd-timesyncd
```

Через несколько минут можно проверить с помощью `timedatectl` состояние синхронизации и дату на сервере

```sh
$ timedatectl status
      Local time: Mon 2020-03-16 09:08:15 MSK
  Universal time: Mon 2020-03-16 06:08:15 UTC
        RTC time: Mon 2020-03-16 06:08:15
       Time zone: Europe/Moscow (MSK, +0300)
     NTP enabled: yes
NTP synchronized: yes
 RTC in local TZ: no
      DST active: n/a

$ date
Mon Mar 16 09:08:17 MSK 2020
```

## UPD 28.04.2020

Для Debian 10 параметр `Server` поменялся на `NTP`

```sh
$ echo 'NTP=192.168.1.1 192.168.1.2' | sudo tee -a /etc/systemd/timesyncd.conf > /dev/null
```

## UPD 23.10.2024

При выполнении команды `sudo timedatectl set-ntp true` появляется ошибка:
```
Failed to set ntp: NTP not supported
```

Решение
```bash
$ sudo apt install systemd-timesyncd
```

## Настройка Chroony

Пришлось настраивать Chroony в Debian на ВМ, где используется централизованная авторизация FreeIPA)

Устанавливаем Chroony (ставится вместе с FreeIPA Client)
```bash
sudo apt update
sudo apt install chrony -y
```

Прописываем наш NTP
```bash
echo 'server 10.20.30.2 iburst' | sudo tee /etc/chrony/sources.d/local-ntp-server.sources
```

Перезапускаем службу
```bash
$ sudo systemctl restart chronyd
```

Проверяем, появился ли наш NTP-сервер в источниках
```bash
$ chronyc sources
MS Name/IP address         Stratum Poll Reach LastRx Last sample               
===============================================================================
^? 185.211.244.47                0   7     0     -     +0ns[   +0ns] +/-    0ns
^? 31.131.251.6                  0   7     0     -     +0ns[   +0ns] +/-    0ns
^? time.cloudflare.com           0   7     0     -     +0ns[   +0ns] +/-    0ns
^? 5.188.119.216                 0   7     0     -     +0ns[   +0ns] +/-    0ns
^? ns1.ooonet.ru                 0   7     0     -     +0ns[   +0ns] +/-    0ns
^? mail.rashnikov.name           0   7     0     -     +0ns[   +0ns] +/-    0ns
^? host198-122.infolink.ru       0   7     0     -     +0ns[   +0ns] +/-    0ns
^* freeipa.itdraft.ru            3   6    17    62   +399ns[  -47us] +/- 3321us
```

В последней строчке видим наш NTP-сервер

Проверяем статус синхронизации времени
```bash
$ timedatectl
               Local time: Wed 2025-01-22 13:45:44 MSK
           Universal time: Wed 2025-01-22 10:45:44 UTC
                 RTC time: Wed 2025-01-22 10:45:44
                Time zone: Europe/Moscow (MSK, +0300)
System clock synchronized: yes
              NTP service: active
          RTC in local TZ: no
```
