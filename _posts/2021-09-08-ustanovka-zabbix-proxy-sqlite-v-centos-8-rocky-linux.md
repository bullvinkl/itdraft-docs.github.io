---
title: "Установка Zabbix Proxy + SQLite в Centos 8 / Rocky Linux"
date: "2021-09-08"
categories: 
  - Linux
  - Zabbix
tags: 
  - "centos"
  - "rocky-linux"
  - "sqlite"
  - "zabbix"
  - "selinux"
image:
  path: /commons/laptop-2.jpg
  alt: "Установка Zabbix Proxy"
---

> **Zabbix proxy** - сервис, способный собирать данные мониторинга с одного или нескольких наблюдаемых устройств и отправлять эту информацию Zabbix серверу, таким образом прокси работает от имени сервера. Используется для масштабирования, централизации Zabbix.

Устанавливаем необходимый софт

```sh
$ sudo dnf install -y zabbix-proxy-sqlite3 zabbix-agent policycoreutils-python-utils nano
```

Создаем каталог для базы данных и назначаем владельца каталога

```sh
$ sudo mkdir /var/lib/zabbix/
$ sudo chown -R zabbix. /var/lib/zabbix/
```

> Если создать каталог отличный от /var/lib/zabbix, в дальнейшем будут проблемы с SELinux
{: .prompt-warning }

Открываем файл настроек zabbix прокси и редактируем его

```sh
$ sudo grep -vE '(^[[:space:]]*([#;!].*)?$)' /etc/zabbix/zabbix_proxy.conf

Server=192.168.31.10     #адрес нашего zabbix сервера
Hostname=srv-zbproxy-01          #имя прокси сервера
LogFile=/var/log/zabbix/zabbix_proxy.log
LogFileSize=1024
PidFile=/var/run/zabbix/zabbix_proxy.pid
SocketDir=/var/run/zabbix
DBName=/var/lib/zabbix/zabbix_proxy /путь до базы данных.
DBUser=zabbix
SNMPTrapperFile=/var/log/snmptrap/snmptrap.log
Timeout=4
LogSlowQueries=3000
StatsAllowedIP=127.0.0.1
```

Запускаем zabbix-proxy (обязательно, что б в дальнейшем сгенерировать правила selinux)

```sh
$ sudo systemctl start zabbix-proxy
```

Сервис не запустится.

Добавляем правила SELinux, что бы он не блокировал работу zabbix proxy

```sh
$ cd /tmp
$ sudo grep zabbix_proxy /var/log/audit/audit.log | grep denied | audit2allow -m zabbix_proxy > zabbix_proxy.te
$ sudo grep zabbix_proxy /var/log/audit/audit.log | grep denied | audit2allow -M zabbix_proxy
$ sudo semodule -i zabbix_proxy.pp
```

Теперь можно запускать zabbix proxy, добавить его в автозагрузку и проверить статус

```sh
$ sudo systemctl start zabbix-proxy
$ sudo systemctl enable zabbix-proxy
$ sudo systemctl status zabbix-proxy
```

Смотрим логи

```sh
$ sudo tail -f /var/log/zabbix/zabbix_proxy.log
```

Настройка PSK шифрования.

Генерируем наш ключ PSK и сохраняем его

```sh
$ openssl rand -hex 32 | sudo tee /var/lib/zabbix/proxy.psk
```

Меняем владельца `proxy.psk`

```sh
$ sudo chown zabbix. /var/lib/zabbix/proxy.psk
```

Редактируем конфиг zabbix proxy, добавляем строки для PSK шифрования

```sh
$ sudo nano /etc/zabbix/zabbix_proxy.conf
...
####### TLS-RELATED PARAMETERS #######
TLSConnect=psk

TLSPSKIdentity=srv-zbproxy-01
TLSPSKFile=/var/lib/zabbix/proxy.psk
```

Перезапускаем сервис

```sh
$ sudo systemctl restart zabbix-proxy
```

Zabbix proxy настроен. Далее его надо прописать на Zabbix Server и там же добавить наш PSK-ключ шифрования