---
title: "Установка почтового сервера iRedMail на CentOS 7. Часть 4. Настройка Postfix, авторизация без ввода домена"
date: "2019-02-09"
categories: 
  - Linux
  - iRedMail
tags: 
  - "centos"
  - "dovecot"
  - "iredmail"
  - "postfix"
  - "roundcube"
  - "sogo"
image:
  path: /commons/bigstock-laptop.jpg
  alt: "Установка iRedMail"
---

> **Postfix** — агент передачи почты (MTA — mail transfer agent). Postfix является свободным программным обеспечением, создавался как альтернатива Sendmail. Изначально Postfix был разработан Вейтсом Венемой в то время, когда он работал в Исследовательском центре имени Томаса Уотсона компании IBM.
{: .prompt-tip }

## Открываем SMTPS (порт 465/tcp)

Редактируем файл конфигурации Postfix - `master.cf`  
Раскомментируем строки

```sh
$ sudo nano /etc/postfix/master.cf
smtps     inet  n       - n       - - smtpd
  -o syslog_name=postfix/smtps
  -o smtpd_tls_wrappermode=yes
  -o smtpd_sasl_auth_enable=yes
  -o smtpd_reject_unlisted_recipient=no
  -o smtpd_client_restrictions=$mua_client_restrictions
  -o smtpd_helo_restrictions=$mua_helo_restrictions
  -o smtpd_sender_restrictions=$mua_sender_restrictions
  -o smtpd_recipient_restrictions=permit_sasl_authenticated,reject
  -o milter_macro_daemon_name=ORIGINATING
```

Перезагружаем Postfix

```sh
$ sudo systemctl restart postfix
```

Добавляем правила в фаерволл

```sh
$ sudo firewall-cmd --permanent --add-service=smtp
$ sudo firewall-cmd --permanent --add-service=pop3
$ sudo firewall-cmd --permanent --add-service=imap
$ sudo firewall-cmd --permanent --add-service=smtps
$ sudo firewall-cmd --permanent --add-service=pop3s
$ sudo firewall-cmd --permanent --add-service=imaps
$ sudo firewall-cmd --reload
```

Проверяем

```sh
$ sudo firewall-cmd --list-all
iredmail (active)
  target: default
  icmp-block-inversion: no
  interfaces: ens192
  sources: 
  services: http https smtp smtp-submission pop3 pop3s imap imaps ssh smtps
  ports: 465/tcp
  protocols: 
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules:
```

## Режем заголовки в Postfix

Для того, что бы не светить свой локальный ip в исходнике письма, отредактируем файл header\_checks, добавив туда строки

```sh
$ sudo nano /etc/postfix/header_checks
/^\s*(Received: from)[^\n]*(.*for <.*@(?!itdraft.ru).*)/ REPLACE $1 [127.0.0.1] (localhost [127.0.0.1])$2
/^\s*Mime-Version: 1.0.*/ REPLACE Mime-Version: 1.0
/^\s*User-Agent/ IGNORE
/^\s*X-Enigmail/ IGNORE
/^\s*X-Mailer/ IGNORE
/^\s*X-Originating-IP/ IGNORE
```

Перезагружаем Postfix

```sh
$ sudo systemctl restart postfix
```

## Авторизация в Dovecot без ввода домена

Редактируем конфигурационный файл dovecot.conf, раскомментируем строку

```sh
$ sudo nano /etc/dovecot/dovecot.conf
auth_default_realm = itdraft.ru
```

Перезагружаем Dovecot

```sh
$ sudo systemctl restart dovecot
```

## Авторизация в Roundcube без ввода домена

Редактируем конфигурационный файл `config.inc.php`, в конце дописать строчку

```sh
$ sudo nano /opt/www/roundcubemail-1.3.8/config/config.inc.php
$config['username_domain'] = 'itdraft.ru';
```

## Авторизация в SOGo без ввода домена

Редактируем конфигурационный файл SOGo `sogo.conf`

```sh
$ sudo nano /etc/sogo/sogo.conf
SOGoLanguage = Russian
SOGoTimeZone = "Europe/Moscow";
SOGoMailDomain = "itdraft.ru";
```

Перезагружаем SOGo

```sh
$ sudo systemctl restart sogod
```

у меня данный метод в `sogo` не заработал, хотя мануал изучал на официальном сайте.
