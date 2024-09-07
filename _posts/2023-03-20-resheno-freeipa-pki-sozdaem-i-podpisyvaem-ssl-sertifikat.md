---
title: "[Решено] FreeIPA PKI - Создаем и подписываем ssl сертификат"
date: "2023-03-20"
categories: 
  - Directory-Service
  - PKI
tags: 
  - "freeipa"
  - "linux"
  - "pki"
  - "ssl"
image:
  path: /commons/cybersec_digest_3.png
  alt: "FreeIPA PKI - Создаем и подписываем ssl сертификат"
---

> **Инфраструктура открытых ключей (PKI)** — набор средств (технических, материальных, и т. д.), распределённых служб и компонентов, используемых для поддержки криптозадач на основе закрытого и открытого ключей.
{: .prompt-tip }

Допустим, у нас развернута служба каталогов FreeIPA в зоне `itdraft.lan`.
Необходимо создать самоподписанный ssl-сертификат и подписывать его средствами FreeIPA. Таким образом сертификат станет доверенным в зоне `itdraft.lan`. А если в организации добавить корневой сертификат FreeIPA в доверенные, то и выпущенные нами ssl-сертификаты тоже будут доверенными.

Добавляем А-запись в DNS FreeIPA

![](/assets/img/posts/2023/03/20/image-14.png){: w="300" }
_Добавляем А-запись_

```
Сетевые службы > DNS > Зоны DNS > itdraft.lan
Имя записи: test
Тип записи: А
IP-адрес: 192.168.1.100
```

Создаем запрос и ключ в терминале

```sh
$ openssl req -batch -new -newkey rsa:2048 -nodes -keyout server.key -subj '/C=RU/ST=Moscow/L=Moscow/O=ITDRAFT.LAN/OU=IT/CN=test.itdraft.lan/emailAddress=admin@itdraft.ru' -out server.csr
```

- Organization Name - наша локальная зона

- Common Name - поддомен в локальной зоне

Создаем службу в админке FreeIPA

![](/assets/img/posts/2023/03/20/image-15.png){: w="300" }
_Создаем службу_

```
Идентификация > Службы > Добавить
Служба: HTTP
Имя узла: test.itdraft.lan
Пропустить проверку узла
```

Далее заходим в созданную службу и добавляем сертификат

![](/assets/img/posts/2023/03/20/image-16.png){: w="300" }
_добавляем сертификат_

```
Действия > Новый сертификат 
Центр сертификации (CA): ipa
В нижнее поле вставить запрос, который мы сгенерировали в терминале (server.csr)
```

Готово, FreeIPA создаст и подпишет сертификат, который будет отображаться в этой же созданной HTTP-службе

![](/assets/img/posts/2023/03/20/image-17.png){: w="300" }
_Проверяем_

Проверяем сертификат в терминале

```sh
$ openssl x509 -noout -modulus -in server.crt | openssl md5
(stdin)= fd7c16f7e585cb4d45b1a5aab79a9397

Проверяем ключ:
$ openssl rsa -noout -modulus -in server.key | openssl md5
(stdin)= fd7c16f7e585cb4d45b1a5aab79a9397

Проверяем запрос:
$ openssl req -noout -modulus -in server.csr | openssl md5
(stdin)= fd7c16f7e585cb4d45b1a5aab79a9397
```

Как видим, хэш-суммы одинаковые
