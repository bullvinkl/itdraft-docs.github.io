---
title: "Не удалось получить доступ к файлу блокировки /var/lib/apt/lists/lock"
date: "2017-06-07"
categories: 
  - Manuals
tags: 
  - "ubuntu"
image:
  path: /commons/913980_54f3_3.jpg
  alt: "Не удалось получить доступ к файлу блокировки"
---

> В Linux-системах, управляемых пакетами с помощью APT (Advanced Package Tool), файл /var/lib/apt/lists/lock играет важную роль. Он блокирует доступ к каталогу списков пакетов (/var/lib/apt/lists) для предотвращения одновременного доступа к нему нескольких процессов.
{: .prompt-tip }

При обновлении Ubuntu 17.04 выскакивает ошибка:

Не удалось получить доступ к файлу блокировки `/var/lib/apt/lists/lock - open (11: Ресурс временно недоступен)`
E: Невозможно заблокировать каталог `/var/lib/apt/lists/`

Ищем виновника:

```sh
$ ps aux | grep '[a]pt'
root     28528  0.0  0.6 293964 104168 ?       SNl  11:42   0:02 /usr/bin/python3 /usr/sbin/aptd
root     28613  0.0  0.5 295084 97948 pts/0    SNs+ 11:43   0:05 /usr/bin/python3 /usr/sbin/aptd
root     28639  0.0  0.3  81220 61780 pts/1    SNs+ 11:43   0:00 /usr/bin/dpkg --status-fd 76 --no-triggers --unpack --auto-deconfigure --recursive /tmp/apt-dpkg-install-CwaS8u
```

Грохаем найденные процессы:

```sh
$ sudo kill 28528
$ sudo kill 28613
$ sudo kill 28639
```

Проверяем:

```sh
$ ps aux | grep '[a]pt'
```

Обновляем систему

```sh
$ sudo apt update && sudo apt upgrade -y
```
