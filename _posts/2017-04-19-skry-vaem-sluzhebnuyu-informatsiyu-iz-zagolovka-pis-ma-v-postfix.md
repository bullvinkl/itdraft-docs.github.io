---
title: "Скрываем служебную информацию из заголовка письма в Postfix"
date: "2017-04-19"
categories: 
  - Communication-System
tags: 
  - "postfix"
image:
  path: /commons/629418_a5b0-1.jpg
  alt: "Скрываем служебную информацию из заголовка письма в Postfix"
---

> **Postfix** имеет модульную архитектуру, позволяющую создать быструю и безопасную почтовую систему. Для открытия порта (TCP 25 порт) требуются привилегии root, а демоны, которые выполняют основную работу, могут работать непривилегированным пользователем в изолированном (chroot) окружении, что позитивно сказывается на безопасности.
{: .prompt-tip }

После настройки Postfix в заголовке почтового сообщения имеем:

```
Received: from [192.168.1.99] (unknown [192.168.1.99]) (using TLSv1.2 with cipher DHE-RSA-AES128-SHA (128/128 bits)) (No client certificate requested) by mx.example.ru (Postfix) with ESMTPS id B7EE3120109 for <xxx@gmail.com>; Thu, 24 Sep 2015 17:18:28 +0300 (MSK)
```

где виден как внешний, так и внутренние IP адреса клиента. А если в конфиге Postfix включена следующая строка:

```
smtpd_sasl_authenticated_header = yes
```

то получатель увидит пользователя почтового сервера из под которого производилась отправка письма. По-хорошему почтовый адрес должен назначаться через алиас, чтобы его имя не соответствовало логину к почтовому серверу.

Исправляем ситуацию, редактируем конфиг Postfix `/etc/postfix/main.cf` и добавляем в него следующую строку:

```
header_checks = pcre:/etc/postfix/auth_header_checks.pcre
```

Создаем файл `/etc/postfix/auth_header_checks.pcre`, добавив в него:

```
/^\s*(Received: from)[^\n]*(.*for <.*@(?!mx.example.ru).*)/ REPLACE $1 [127.0.0.1] (localhost [127.0.0.1])$2 /^\s*Mime-Version: 1.0.*/ REPLACE Mime-Version: 1.0 /^\s*User-Agent/ IGNORE /^\s*X-Enigmail/ IGNORE /^\s*X-Mailer/ IGNORE /^\s*X-Originating-IP/ IGNORE
```

Перезапускаем службу:

```sh
$ sudo service postfix restart
```

и проверяем, отправив себе тестовое письмо:

```
Received: from [127.0.0.1] (localhost [127.0.0.1]) by mx.example.ru (Postfix) with ESMTPS id B7EE3120109 for <xxx@gmail.com>; Thu, 24 Sep 2015 17:18:28 +0300 (MSK)
```
