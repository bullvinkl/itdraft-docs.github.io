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
