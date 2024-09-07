---
title: "Установка и настройка сервера FreeIPA в Centos 7"
date: "2019-11-11"
categories: 
  - Linux
  - FreeIPA
tags: 
  - "centos"
  - "freeipa"
image:
  path: /commons/virtual-reality-development.jpg
  alt: "Установка и настройка FreeIPA"
---

> **FreeIPA** — это комплексное решение для централизованного управления безопасностью Linux-систем, идентификацией и аутентификацией. Оно позволяет создавать многоуровневую систему управления доступом, где руководители подразделений могут добавлять пользователей в группы и настраивать доступ к ресурсам.
{: .prompt-tip }

В программе FreeIPA для управления идентификацией серверов используются доменные имена. Так же FreeIPA можно использовать как панель управления NS-записями

## Подготовительный этап, синхронизация с NTP-сервером

Для синхронизации времени рекомендуется использовать NTP  
Остановим и выключим `chrony`, если он предустановлен

```sh
$ sudo systemctl stop chronyd
$ sudo systemctl disable chronyd
```

Установим NTP и настроим синхронизацию с нашим ntp-сервером

```sh
$ sudo yum install ntpdate
$ sudo nano /etc/ntp.conf
server 192.168.1.10
```

Смотрим расхождение во времени:

```sh
$ sudo ntpdate -qu 192.168.1.10
server 192.168.1.10, stratum 3, offset -14.866896, delay 0.04178
11 Nov 09:46:42 ntpdate[19110]: step time server 192.168.1.10 offset -14.866896 sec
```

Останавливаем службу, синхронизируем время, запускаем службу

```sh
$ sudo systemctl stop ntpd
$ sudo ntpdate 192.168.1.10
$ sudo systemctl start ntpd
```

Активируем NTP клиента, и смотрим статус

```sh
$ sudo timedatectl set-ntp true
$ sudo timedatectl status
      Local time: Mon 2019-11-11 09:49:10 MSK
  Universal time: Mon 2019-11-11 06:49:10 UTC
        RTC time: Mon 2019-11-11 06:49:10
       Time zone: Europe/Moscow (MSK, +0300)
     NTP enabled: yes
NTP synchronized: yes
 RTC in local TZ: no
      DST active: n/a
```

Можно еще раз проверить расхождение во времени:

```sh
$ sudo ntpdate -qu 192.168.1.10
server 192.168.1.10, stratum 3, offset 0.003698, delay 0.04181
11 Nov 09:49:45 ntpdate[19143]: adjust time server 192.168.1.10 offset 0.003698 sec
```

## Настройка доменного имени сервера и сети

Имя FreeIPA сервера должно быть полным (FQDN), установим его:

```sh
$ sudo hostnamectl set-hostname srv-ipa-01.domain.local
$ sudo hostnamectl status
   Static hostname: srv-ipa-01.domain.local
...
```

Пропишем адрес сервера FreeIPA в файле `/etc/hosts`  
Если мы планируем добавлять реплицирующий FreeIPA сервер, то его имя и IP-адрес так же надо добавить

```sh
$ sudo nano /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.1.11   srv-ipa-01.domain.local srv-ipa-01
192.168.1.12   srv-ipa-02.domain.local srv-ipa-02
```

Если в сети планируется использовать короткие имена машин, должна быть прописана корректная опция `domain` или `search`.

Если мы будем использовать FreeIPA-сервер как DNS, припишем в файле `resolve.conf` его ip-адрес (и ip-адрес реплицирующего, если планируется репликация)

```sh
$ sudo nano /etc/resolve.conf 
search domain.local
# DNS FreeIPA
nameserver 192.168.1.11
nameserver 192.168.1.11
# Наши DNS
nameserver 192.168.1.1
nameserver 192.168.1.2
```

Отредактируем файл с настройками сетевого интерфейса.  
Параметр `PEERDNS="no"` - отвечает, чтобы файл `resolve.conf` не перезаписывался, после перезапуска службы Network

```sh
$ sudo nano /etc/sysconfig/network-scripts/ifcfg-ens160
...
IPADDR="192.168.1.11"
PREFIX="24"
GATEWAY="192.168.1.1"
DNS1="192.168.1.1"
DNS2="192.168.1.2"
PEERDNS="no"
DOMAIN="domain.local"
...
```

## Установка программного обеспечения

Устанавливаем необходимые пакеты

```sh
$ sudo yum -y install bind bind-utils bind-dyndb-ldap ipa-server ipa-client ipa-server-dns
```

FreeIPA требует большого количества случайных данных для выполняемых им криптографических операций. Установим генератор случайных чисел `rngd`

```sh
$ sudo yum install rng-tools
$ sudo systemctl start rngd
$ sudo systemctl enable rngd
```

## Установка и настройка FreeIPA

Установим FreeIPA

```sh
$ sudo ipa-server-install
```

* * *

При установки, можно включить опцию создания директорий пользователей

```sh
$ ipa-server-install --mkhomedir
```

Если вы ее не указали, то эту опцию можно добавить после установки FreeIPA сервера командой:

```sh
$ sudo authconfig --enablemkhomedir --update
```

Если мы не планируем использовать FreeIPA в качестве DNS, то на соответствующий вопрос отвечаем `NO`, и дальше устанавливаем параметры

```
Do you want to configure integrated DNS (BIND)? [no]: no

Server host name [srv-ipa-01.domain.local]: srv-ipa-01.domain.local
Please confirm the domain name [domain.local]: srv-ipa-01.domain.local
Please provide a realm name [SRV-IPA-01.DOMAIN.LOCAL]: SRV-IPA-01.DOMAIN.LOCAL
Directory Manager password:
IPA admin password:

Continue to configure the system with these values? [no]: yes
```

Если мы планируем использовать FreeIPA в качестве DNS, то на соответствующий вопрос отвечаем `YES`, и дальше устанавливаем параметры

```
Do you want to configure integrated DNS (BIND)? [no]: yes

Server host name [srv-ipa-01.domain.local]: srv-ipa-01.domain.local
Please confirm the domain name [domain.local]: domain.local
Please provide a realm name [DOMAIN.LOCAL]: DOMAIN.LOCAL
Directory Manager password:
IPA admin password:
```

Далее пойдут вопросы по настройке DNS

Если мы не хотим перенаправлять DNS запросы с FreeIPA на другой сервер, в соответствующем вопросе пишем `NO`

```
Checking DNS domain domain.local., please wait ...
Do you want to configure DNS forwarders? [yes]: no
No DNS forwarders configured
```

Если мы хотим перенаправлять DNS запросы с FreeIPA на другой сервер, в соответствующем вопросе пишем `YES`, и далее оставляем DNS, которые предложит FreeIPA, либо прописываем другие

```
Checking DNS domain domain.local., please wait ...
Do you want to configure DNS forwarders? [yes]:
Following DNS servers are configured in /etc/resolv.conf: 192.168.1.11, 192.168.1.12
Do you want to configure these servers as DNS forwarders? [yes]:
All DNS servers from /etc/resolv.conf were added. You can enter additional addresses now:
Enter an IP address for a DNS forwarder, or press Enter to skip: 8.8.8.8
DNS forwarder 8.8.8.8 added. You may add another.
Enter an IP address for a DNS forwarder, or press Enter to skip: 8.8.4.4
DNS forwarder 8.8.4.4 added. You may add another.
Enter an IP address for a DNS forwarder, or press Enter to skip:
Checking DNS forwarders, please wait…
```

Разрешаем обратную зону

```
Do you want to search for missing reverse zones? [yes]:
Do you want to create reverse zone for IP 192.168.1.11 [yes]:
Please specify the reverse zone name [1.168.192.in-addr.arpa.]:
Using reverse zone(s) 1.168.192.in-addr.arpa.
```

Далее отобразятся наши данные для проверки

```
The IPA Master Server will be configured with:
Hostname:	srv-ipa-01.domain.local
IP address(es):	192.168.1.11
Domain name:	domain.local
Realm name: 	DOMAIN.LOCAL

BIND DNS server will be configured to serve IPA domain with:
Forwarders:	8.8.8.8, 8.8.4.4
Forward policy:	only
Reverse zone(s):  1.168.192.in-addr.arpa.
Continue to configure the system with these values? [no]: yes
```

Затем пойдет долгий процесс установки, и отобразятся какие порты надо открыть

```
1. You must make sure these network ports are open:
TCP Ports:
* 80, 443: HTTP/HTTPS
* 389, 636: LDAP/LDAPS
* 88, 464: kerberos
* 53: bind
UDP Ports:
* 88, 464: kerberos
* 53: bind
* 123: ntp
```

Открываем эти порты

```sh
$ sudo firewall-cmd --permanent --zone=public --add-service={ntp,http,https,ldap,ldaps,kerberos,kpasswd,dns}
$ sudo firewall-cmd --permanent --zone=public --add-port=53/tcp
$ sudo firewall-cmd --permanent --zone=public --add-port=53/udp
$ sudo firewall-cmd --reload
```

## Проверка

Получаем билет kerberos

```sh
$ kinit admin
Password for admin@DOMAIN.LOCAL:
```

Проверим полученный билет

```sh
$ klist
Ticket cache: KEYRING:persistent:1000:1000
Default principal: admin@DOMAIN.LOCAL

Valid starting       Expires              Service principal
11/05/2019 15:55:28  11/06/2019 15:55:20  krbtgt/DOMAIN.LOCAL@DOMAIN.LOCAL
```

Если на каком-нибудь этапе возникают ошибка, FreeIPA-сервер можно удалить, и попытаться установить заново

```sh
$ sudo ipa-server-install --uninstall -U
```

Что бы зайти в вэб-интерфейс, надо прописать ip-адрес и доменное имя в файле hosts, и в браузере набрать доменное имя
