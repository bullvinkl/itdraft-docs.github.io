---
title: "Установка почтового сервера iRedMail на CentOS 7. Часть 6. DKIM, SPF, DMARC"
date: "2019-02-11"
categories: 
  - Linux
  - iRedMail
tags: 
  - "centos"
  - "dkim"
  - "dmarc"
  - "iredmail"
  - "spf"
image:
  path: /commons/1292838_f55d.jpg
  alt: "Установка iRedMail"
---

> **DKIM**, DomainKeys Identified Mail — метод E-mail аутентификации, разработанный для обнаружения подделывания сообщений, пересылаемых по email. Метод дает возможность получателю проверить, что письмо действительно было отправлено с заявленного домена. DKIM упрощает борьбу с поддельными адресами отправителей, которые часто используются в фишинговых письмах и в почтовом спаме.  
> Технология DomainKeys Identified Mail (DKIM) объединяет несколько существующих методов антифишинга и антиспама с целью повышения качества классификации и идентификации легитимной электронной почты. Вместо традиционного IP-адреса, для определения отправителя сообщения DKIM добавляет в него цифровую подпись, связанную с именем домена организации. Подпись автоматически проверяется на стороне получателя, после чего, для определения репутации отправителя, применяются «белые списки» и «чёрные списки».

## Настраиваем DKIM

Смотрим наше значение DKIM которое сгенерировалось после установки iRedMail

```sh
$ sudo amavisd -c /etc/amavisd/amavisd.conf showkeys
; key#1 1024 bits, i=dkim, d=example.com, /var/lib/dkim/example.com.pem
dkim._domainkey.example.com.	3600 TXT (
  "v=DKIM1; p="
"MIGfMA0GCSqGSIb5RTEBAQUAA4GNADCBiQKBgQCeZ7mcAV0oqaAXOYBOaEMjJHCC"
"SC9+dJbJEwt0KTZpZFAKmOQiZ5h5xzW6PsnGjAXiA6qYEB+xW4KRLOPI35L4h2/U"
"81ppX6St/GhUYIXjV/FB6bBf9I6YgNUzJi549VWnBgo3yNIRgWzQjqounF6wsmAd"
"VQk0V+YoL9FA7qcj2wIDATRE")
```

В панели управления зонами доменного имени создаем TXT-запись:

```
Домен: dkim._domainkey.example.com.
Тип записи: TXT
Значение: v=DKIM1; p=MI...
```

Проверяем

```sh
$ sudo amavisd -c /etc/amavisd/amavisd.conf testkeys
TESTING#1 example.com: dkim._domainkey.example.com => pass
```

## Настраиваем SPF

> **SPF**, Sender Policy Framework (инфраструктура политики отправителя) — расширение для протокола отправки электронной почты через SMTP.  
> Благодаря SPF можно проверить, не подделан ли домен отправителя.

В панели управления зонами доменного имени создаем TXT-запись:

```
Домен: example.com.
Тип записи: TXT
Значение: v=spf1 a mx ip4:%ip% include:_spf.google.com ~all
```

где  
- %ip%` - ip адрес почтового сервера,  
- `include:_spf.google.com` - что бы пройти проверку сервисом гугла, не обязательно прописывать  
- `~all` - принимать письма со всех остальных серверов, но помечать их как СПАМ

## Настраиваем DMARC

> **DMARC**, Domain-based Message Authentication, Reporting and Conformance (идентификация сообщений, создание отчётов и определение соответствия по доменному имени) — это техническая спецификация, созданная группой организаций, предназначенная для снижения количества спамовых и фишинговых электронных писем, основанная на идентификации почтовых доменов отправителя на основании правил и признаков, заданных на почтовом сервере получателя.

Для получения DMARC-отчетов я пользуюсь сервисом [Postmark](https://dmarc.postmarkapp.com/)

Переходим по ссылке выше, указываем свой `e-mail` и `доменное имя`, после чего сервис выдает какое значение надо прописать в панели управления зонами, например:

```
Домен: _dmarc.example.com.
Тип записи: TXT
Значение: v=DMARC1; p=none; pct=100; rua=mailto:абракадабра@dmarc.postmarkapp.com; sp=none; aspf=r;
```

После того, как прописали это значение, надо его подтвердить в сервисе.

## Сервисы для тестирования почтового сервера

[https://postmaster.google.com/](https://postmaster.google.com/)  
[https://www.mail-tester.com/](https://www.mail-tester.com/)  
[https://toolbox.googleapps.com/apps/checkmx/](https://toolbox.googleapps.com/apps/checkmx/)  
[https://dnschecker.org/](https://dnschecker.org/)