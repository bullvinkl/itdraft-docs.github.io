---
title: "Синхронизация времени в Centos 7"
date: "2018-12-20"
categories: 
  - Manuals
tags: 
  - "centos"
  - "chroony"
  - "ntp"
  - "ntpdate"
image:
  path: /commons/986254_f32a_2.jpg
  alt: "Синхронизация времени в Centos"
---

> **NTP** (англ. Network Time Protocol — протокол сетевого времени) — сетевой протокол для синхронизации внутренних часов компьютера с использованием сетей с переменной латентностью. Протокол был разработан Дэвидом Л. Миллсом, профессором Делавэрского университета, в 1985 году.
{: .prompt-tip }

Рассмотрим 2 утилиты синхронизации времени в Centos:

- `ntp / ntpdate`
- `chroony`

## Синхронизация времени через ntp

Установим софт из стандартного репозитория

```sh
$ sudo yum install ntp ntpdate
```

Выполним синхронизацию вручную

```sh
$ sudo ntpdate 1.ru.pool.ntp.org
20 Dec 12:07:40 ntpdate[20628]: adjust time server 85.21.78.91 offset -0.000150 sec
```

Чтобы просто запросить сервер и не устанавливать часы выполним команду ntpdate со следующими флагами

```sh
$ sudo ntpdate -qu 1.ru.pool.ntp.org
server 80.240.216.155, stratum 2, offset 0.000983, delay 0.02901
server 85.21.78.8, stratum 2, offset -0.000851, delay 0.02788
server 89.175.20.7, stratum 1, offset 0.000247, delay 0.02930
server 195.210.189.106, stratum 1, offset 0.000085, delay 0.03043
20 Dec 12:12:03 ntpdate[20717]: adjust time server 89.175.20.7 offset 0.000247 sec
```

Для установки нужных серверов синхронизации времени отредактируем файл `ntp.conf`, и вместо серверов по-умолчанию можно прописать нужные

```sh
$ sudo nano /etc/ntp.conf
...
server 0.centos.pool.ntp.org iburst
server 1.centos.pool.ntp.org iburst
server 2.centos.pool.ntp.org iburst
server 3.centos.pool.ntp.org iburst
...
```

Активируем NTP client и проверим статус

```sh
$ sudo timedatectl set-ntp true 
$ timedatectl status
      Local time: Чт 2018-12-20 12:19:23 MSK
  Universal time: Чт 2018-12-20 09:19:23 UTC
        RTC time: Чт 2018-12-20 09:19:23
       Time zone: Europe/Moscow (MSK, +0300)
     NTP enabled: yes
NTP synchronized: yes
 RTC in local TZ: no
      DST active: n/a
```

Для проверки системных часов надо ввести команду `date`

```sh
$ date
Чт дек 20 12:20:29 MSK 2018
```

## Синхронизация времени через chroony

По-умолчанию в Centos 7 minimal синхронизация времени не настроена

```sh
$ timedatectl
      Local time: Чт 2018-12-20 12:00:35 MSK
  Universal time: Чт 2018-12-20 09:00:35 UTC
        RTC time: Чт 2018-12-20 08:59:21
       Time zone: Europe/Moscow (MSK, +0300)
     NTP enabled: n/a
NTP synchronized: no
 RTC in local TZ: no
      DST active: n/a
```

Установим софт из стандартного репозитория

```sh
$ sudo yum install chrony
```

Для изменения серверов синхронизации времени надо отредактировать файл` /etc/chrony.conf`

```sh
$ sudo nano /etc/chrony.conf
# Use public servers from the pool.ntp.org project.
# Please consider joining the pool (http://www.pool.ntp.org/join.html).
server 0.centos.pool.ntp.org iburst
server 1.centos.pool.ntp.org iburst
server 2.centos.pool.ntp.org iburst
server 3.centos.pool.ntp.org iburst
```

Запускаем сервис и добавляем его в автозагрузку

```sh
$ sudo systemctl start chronyd
$ sudo systemctl enable chronyd
```

Смотрим статус

```sh
$ chronyc sources
210 Number of sources = 4
MS Name/IP address         Stratum Poll Reach LastRx Last sample               
===============================================================================
^* bagnikita.com                 2   6    17     1   -401us[ -730us] +/- 26ms
^+ lhr1.m-d.net                  2   6    17     0   +600us[ +600us] +/- 64ms
^- tor-relais2.link38.eu         2   6    17     1   +392us[ +392us] +/- 34ms
^- ntp.truenetwork.ru            2   6    17     2   -228us[ -558us] +/- 104ms
```

Проверяем, активировалась ли синхронизация

```sh
$ timedatectl
      Local time: Чт 2018-12-20 12:01:28 MSK
  Universal time: Чт 2018-12-20 09:01:28 UTC
        RTC time: Чт 2018-12-20 09:01:28
       Time zone: Europe/Moscow (MSK, +0300)
     NTP enabled: yes
NTP synchronized: yes
 RTC in local TZ: no
      DST active: n/a
```
