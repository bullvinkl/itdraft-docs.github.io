---
layout: post
title: "Установка и настройка Fail2Ban в Centos 7"
date: "2019-04-10"
categories:
  - Linux
  - Fail2Ban
tags:
  - centos
  - fail2ban
image:
  path: /commons/913980_54f3_3.jpg
  alt: "Защита от брутфорса админки Wordpress в Docker с помощью Fail2Ban"
---

> **Fail2Ban** - это программная среда, предназначенная для предотвращения вторжений, защищающая серверы от атак методом перебора.  
> Fail2Ban работает, отслеживая файлы журналов (например, /var/log/auth.log, /var/log/apache/access.log и т. Д.) для выбранных сервисов и запускает сценарии на их основе. Чаще всего используется для блокировки выбранных IP-адресов, которые могут принадлежать хостам, которые пытаются нарушить безопасность системы.

## Установка Fail2Ban

Добавляем в систему репозиторий EPEL, обновляемся и устанавливаем программное обеспечение
```sh
# yum install epel-release
# yum update
# yum install fail2ban fail2ban-systemd
```

Если у вас не отключен SELinux, то обновляем политики
```sh
# yum update -y selinux-policy*
```

## Настройка Fail2Ban

Fail2Ban хранит свои настройки в файле `/etc/fail2ban/jail.conf`, и при обновлении перезаписывает этот файл, по-этому скопируем файл jail.conf под именем jail.local
```sh
# cp -pf /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
```

По-умолчанию файл конфигурации содержит следующие строки:
```sh
[DEFAULT]

#
# MISCELLANEOUS OPTIONS
#

# "ignoreip" can be an IP address, a CIDR mask or a DNS host. Fail2ban will not
# ban a host which matches an address in this list. Several addresses can be
# defined using space separator.
ignoreip = 127.0.0.1/8

# External command that will take an tagged arguments to ignore, e.g. <ip>,
# and return true if the IP is to be ignored. False otherwise.
#
# ignorecommand = /path/to/command <ip>
ignorecommand =

# "bantime" is the number of seconds that a host is banned.
bantime = 600

# A host is banned if it has generated "maxretry" during the last "findtime"
# seconds.
findtime = 600

# "maxretry" is the number of failures before a host get banned.
maxretry = 5
```

- `ignoreip` - используется для установки списка IP-адресов, которые не будут забанены. Список IP-адресов следует указывать через пробел.
- `bantime` - время блокировки, в секундах.
- `maxretry` - количество попыток перед перед блокировкой.
- `findtime` - время, на протяжении которого рассчитывается количество попыток перед баном (maxretry).

Т.е. в конфиге по-умолчанию прописано, что пользователь будет забанен на 10 минут, если в течение 10 минут будет совершено 5 неудачных попыток.

## Защита SSH соединения

Создадим новый файл sshd.local, и добавим в него следующие строки

```sh
# nano /etc/fail2ban/jail.d/sshd.local

[sshd]
enabled = true
port = ssh
action = firewallcmd-ipset
logpath = %(sshd_log)s
maxretry = 5
bantime = 86400
```

- `enable = true` - проверка ssh активна.
- `action` используется для получения IP-адреса, который необходимо заблокировать, используя фильтр, доступный в /etc/fail2ban/action.d/firewallcmd-ipset.conf.
- `logpath` - путь, где хранится файл журнала. Этот файл журнала сканируется Fail2Ban.
- `maxretry` - лимита для неудачных входов в систему.
- `bantime` - время блокировки (24 часа).

Добавим сервис Fail2Ban в автозагрузку и запустим его
```sh
# systemctl enable fail2ban
# systemctl start fail2ban
```

Что бы проверить статус,выполним команду
```sh
# fail2ban-client status

Status
|- Number of jail: 1
`- Jail list: sshd
```

Для разблокирования IP-адреса надо выполнить команду
```sh
# fail2ban-client set sshd unbanip IPADDRESS
```