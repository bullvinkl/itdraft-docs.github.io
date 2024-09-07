---
title: "Отказоустойчивость, настройка репликации FreeIPA на Centos 7"
date: "2019-11-12"
categories: 
  - Directory-Service
tags: 
  - "centos"
  - "freeipa"
  - "replication"
image:
  path: /commons/bigstock-Concept-Of-Data-Network-Manage-246960574_0.jpg
  alt: "Репликация FreeIPA"
---

> **Репликация** — это процесс, под которым понимается копирование данных из одного источника на другой (или на множество других) и наоборот. При репликации изменения, сделанные в одной копии объекта, могут быть распространены в другие копии.
{: .prompt-tip }

## Подготовка сервера

Как и в [предыдущей статье]({% post_url 2019-11-11-ustanovka-i-nastrojka-servera-freeipa-v-centos-7 %}), надо:

- настроить синхронизацию с NTP-сервером
- добавить в /etc/hosts записи FreeIPA-серверов
- прописать в /etc/resolve.conf ip-адреса FreeIPA-серверов

Имя FreeIPA сервера должно быть полным (FQDN), установим его:

```sh
$ sudo hostnamectl set-hostname srv-ipa-02.domain.local
$ sudo hostnamectl status
   Static hostname: srv-ipa-02.domain.local
...
```

Установим необходимый софт:

```sh
$ sudo yum install bind-utils ipa-client ipa-server-dns
```

## Настройка репликации

Настраиваем FreeIPA client, запустим этот процесс с нашими параметрами

```sh
$ sudo ipa-client-install \
 --mkhomedir --domain="domain.local" \
 --server="srv-ipa-01.domain.local" \
 --server="srv-ipa-02.domain.local" \
 --realm="DOMAIN.LOCAL" \
 --principal="admin" \
 --password="%PASSWORD%" \
 --enable-dns-updates -U \
 --force-join \
 --force-ntpd
```

Запустим процесс создания реплики

```sh
$ sudo ipa-replica-install
Password for admin@DOMAIN.LOCAL:
```

```sh
$ sudo ipa-ca-install
Directory Manager (existing master) password:
```

Если мы используем FreeIPA в качестве DNS, запустим процесс установки dns-сервера

```sh
$ sudo ipa-dns-install
Do you want to configure DNS forwarders? [yes]: yes
Do you want to search for missing reverse zones? [yes]: yes
```

Настраиваем firewall, открываем необходимые порты

```sh
$ sudo firewall-cmd --permanent --zone=public --add-service={ntp,http,https,ldap,ldaps,kerberos,kpasswd,dns}
$ sudo firewall-cmd --permanent --zone=public --add-port=53/tcp
$ sudo firewall-cmd --permanent --zone=public --add-port=53/udp
$ sudo firewall-cmd --reload
```

## Проверка

Что бы проверить, добавился ли реплицирующий сервер, заходим в web-интерфейс основного или реплицирующего FreeIPA сервера (надо прописать ip-адрес и доменное имя в файле hosts)

```
IPA Servers - Topology - Topology Graph
```
