---
title: "iptables открываем 80 порт"
date: "2015-07-31"
categories: 
  - Security-System
tags: 
  - "centos"
  - "iptables"
image:
  path: /commons/recommendation_sys-min.png
  alt: "открываем 80 порт"
---

> **iptables** - утилита командной строки, является стандартным интерфейсом управления работой межсетевого экрана (брандмауэра) netfilter для ядер Linux, начиная с версии 2.4. Для использования утилиты iptables требуются привилегии суперпользователя (root).
{: .prompt-tip }

Редактируем конфигурационный файл:

```sh
$ sudo nano /etc/sysconfig/iptables
```

Добавляем строчку

```
-A INPUT -m state --state NEW -m tcp -p tcp --dport 80 -j ACCEPT
```

Перезапускаем службу iptables

```sh
$ sudo service iptables restart
```
