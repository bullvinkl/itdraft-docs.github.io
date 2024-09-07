---
title: "Установка и настройка DKIM на CentOS"
date: "2017-04-17"
categories: 
  - Communication-System
tags: 
  - "centos"
  - "dkim"
  - "opendkim"
  - "postfix"
image:
  path: /commons/424864_ab6e.jpg
  alt: "Установка и настройка DKIM"
---

> **DKIM** (DomainKeys Identified Mail) - это стандарт защиты электронной почты, который обеспечивает подлинность отправителя электронного письма. Это достигается за счет добавления специального криптографического ключа в заголовок письма, который позволяет серверу-получателю проверять authenticity отправителя.
{: .prompt-tip }

Устанавливаем opendkim

```sh
$ sudo yum -y install opendkim
$ sudo mkdir -p /etc/opendkim/keys
$ sudo chown -R opendkim:opendkim /etc/opendkim
$ sudo chmod -R go-wrx /etc/opendkim/keys
```

Приводим конфигурационный файл opendkim к виду: 

```sh
$ sudo nano /etc/opendkim.conf
AutoRestart Yes
AutoRestartRate 10/1h
PidFile /var/run/opendkim/opendkim.pid
Mode sv
Syslog yes
SyslogSuccess yes
#LogWhy yes
UserID opendkim:opendkim
Socket inet:8891@localhost
Umask 022
Canonicalization relaxed/relaxed
Selector default
Background yes
MinimumKeyBits 1024
KeyFile /etc/opendkim/keys/example.ru/default
KeyTable /etc/opendkim/KeyTable
SigningTable refile:/etc/opendkim/SigningTable
ExternalIgnoreList refile:/etc/opendkim/TrustedHosts
InternalHosts refile:/etc/opendkim/TrustedHosts
```

Перегружаем postfix и opendkim

```sh
$ sudo hash -r
$ sudo service opendkim restart
$ sudo service postfix restart
```

## Настраиваем почтовый домен, например itdraft.ru:

Создаем каталог, генерируем ключ с помощью утилиты `opendkim-genkey`

```sh
$ sudo mkdir /etc/opendkim/keys/itdraft.ru
$ sudo opendkim-genkey -D /etc/opendkim/keys/itdraft.ru/ -d itdraft.ru -s default
```

Меняем владельца на сгенерированный ключ и переименовываем его

```sh
$ sudo chown -R opendkim:opendkim /etc/opendkim/keys/itdraft.ru
$ sudo mv /etc/opendkim/keys/itdraft.ru/default.private /etc/opendkim/keys/itdraft.ru/default
```

Добавляем правило для нашего домен в файл со списком ключей, доступных для подписи (/etc/opendkim/KeyTable)

```sh
$ sudo echo -e "default._domainkey.itdraft.ru itdraft.ru:default:/etc/opendkim/keys/itdraft.ru/default" >> /etc/opendkim/KeyTable
```

Добавляем правило для нашего домен в файл со списком доменов и аккаунтов доступных для подписи (/etc/opendkim/SingleTable)

```sh
$ sudo echo -e "*@itdraft.ru default._domainkey.itdraft.ru" >> /etc/opendkim/SigningTable
```

Добавляем наши домены в файл со списком доверенных доменов при подписывании или проверке (/etc/opendkim/TrustedHosts)

```sh
$ sudo echo -e "itdraft.ru\mx.itdraft.ru" >> /etc/opendkim/TrustedHosts
```

Перегружаем postfix и opendkim

```sh
$ sudo hash -r
$ sudo service opendkim restart
$ sudo service postfix restart
```

При генерации сертификата, утилита `opendkim-genkey` создала файл с данными, которые надо прописать в наш DNS

```sh
$ sudo nano /etc/opendkim/keys/itdraft.ru/default.txt
default._domainkey IN TXT v=DKIM1; k=rsa; p=MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQC3kbEHhBnq478wOR6AtcG8VND9ObsxdnBvKc4tRaEGaTdDz9xuK/YXxQUJ4TuOSetnUo4lbnyod8sGddUYJYDB84PZAQVQsRYW5hlaOOrjisEE+ph85gXvZnLQ+l6KLrTWHh4GlWx4UexclK9eQ+wXc/9kl9Yow6+9/gmDe/eRnQIDAQAB;
```
