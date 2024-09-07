---
layout: post
title: "[Решено] Защита от брутфорса админки Wordpress в Docker с помощью Fail2Ban"
date: "2023-09-04"
categories:
 - Linux
 - Docker
 - Fail2Ban
tags:
 - wordpress
 - docker
 - fail2ban
image:
  path: /commons/no-release-repo.jpg
  alt: "Защита от брутфорса админки Wordpress в Docker с помощью Fail2Ban"
---

> Fail2Ban – это программная среда, предназначенная для предотвращения вторжений, защищающая серверы от атак методом перебора.  
> Fail2Ban работает, отслеживая файлы журналов (например, /var/log/auth.log, /var/log/apache/access.log и т. Д.) для выбранных сервисов и запускает сценарии на их основе. Чаще всего используется для блокировки выбранных IP-адресов, которые могут принадлежать хостам, которые пытаются нарушить безопасность системы.
{: .prompt-tip }

Установка Fail2Ban была рассмотрена в [одной из предыдущих статей]({% post_url 2019-04-10-ustanovka-i-nastrojka-fail2ban-v-centos-7 %})

Для блокирования ip недоброжелателей от брутфорса админки Wordpress, развернутом в docker исполнении, Fail2Ban парсит `access.log` файл.  
Можно в переменной logpath указать путь до логов контейнеров:
```sh
...
logpath = /var/lib/docker/containers/*/*-json.log
...
```

Я вынес этот файл из контейнера и примонтировал к каталогу с проектом.

Редактируем конфигурационный файл Fail2Ban с нашими правилами (либо дефолтный `/etc/fail2ban/jail.local`, либо который мы создали в каталоге `/etc/fail2ban/jail.d`)
```sh
$ sudo nano /etc/fail2ban/jail.d/defaults-debian.conf
...
[wp-login]
enabled  = true
port     = http,https
filter   = wp-login
chain    = DOCKER-USER
logpath  = /opt/itdraft.ru/logs/access.log
bantime  = 604800
findtime = 7200
maxretry = 3
```

Основное отличие от wordpress не в docker исполнении - наличии строки:
```
...
chain    = DOCKER-USER
...
```

Создаем фильтр под wordpress
```sh
$ sudo nano /etc/fail2ban/filter.d/wp-login.conf
[Definition]
failregex = <HOST>.*POST.*(wp-login.php|xmlrpc.php).* 200
ignoreregex =
```

Перезапускам Fail2Ban
```sh
$ sudo systemctl restart fail2ban
```

Проверить работу фильтра
```sh
$ sudo fail2ban-client status wp-login
$ sudo iptables -L -v -n
$ sudo less /var/log/fail2ban.log
```
