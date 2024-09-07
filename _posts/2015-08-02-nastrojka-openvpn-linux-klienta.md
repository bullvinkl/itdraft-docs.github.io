---
title: "Настройка OpenVPN Linux клиента"
date: "2015-08-02"
categories: 
  - Security-System
tags: 
  - "centos"
  - "linux"
  - "openvpn"
image:
  path: /commons/open-source-proprietary.png
  alt: "OpenVPN клиент"
---

> **OpenVPN** - бесплатное программное решение для реализации подключений по протоколу VPN к виртуальным частным сетям, создания шифрованных туннелей между сервером и клиентскими компьютерами или подключения типа точка-точка для безопасной передачи данных через Интернет.
{: .prompt-tip }

Подключаем репозиторий EPEL откуда мы поставим OpenVPN.  

```sh
$ sudo rpm -Uvh http://download.fedora.redhat.com/pub/epel/5/i386/epel-release-5-4.noarch.rpm
```

Устанавливаем OpenVPN  

```sh
$ sudo yum -y install openvpn
```

Добавляем OpenVPN в автозагрузку при старте компьютера  

```sh
$ sudo chkconfig openvpn on
```

В каталог `/etc/openvpn/` копируем шаблон файла конфигурации OpenVPN-клиента  

```sh
$ sudo cp /usr/share/doc/openvpn-2.1/sample-config-files/client.conf /etc/openvpn/
```

И приводим его к следующему виду:

```sh
$ sudo nano /etc/openvpn/client.conf
client  
dev tun  
proto tcp  
remote 194.28.85.220 1194  
resolv-retry infi nite  
user nobody  
group nobody  
persist-key  
persist-tun  
ca ca.crt  
cert pc1.crt  
key pc1.key  
ns-cert-type server  
log-append /var/log/openvpn.log  
comp-lzo  
verb 3
```

Теперь копируем файлы `ca.crt`, `pc1.crt` и `pc1.key` с OpenVPN сервера в папку `/etc/openvpn/`

```sh
$ sudo cd /etc/openvpn/  
$ sudo scp root@194.28.85.220:/etc/openvpn/ca.crt ./  
$ sudo scp root@194.28.85.220:/etc/openvpn/2.0/keys/pc1.key ./  
$ sudo scp root@194.28.85.220:/etc/openvpn/2.0/keys/pc1.crt ./
```

Где
- `194.28.85.220` - IP-адрес сервера.

Запускаем OpenVPN:

```sh
$ sudo service openvpn start
```
