---
title: "Проверка соответствия ключа и ssl сертификата"
date: "2018-09-20"
categories: 
  - Manuals
tags: 
  - "md5"
  - "ssl"
image:
  path: /commons/1039062_e553_4.jpg
  alt: "Проверка соответствия ключа и ssl сертификата"
---

> **SSL-сертификат** (Secure Sockets Layer) - это цифровой документ, который подтверждает надежность веб-сайта и обеспечивает безопасную передачу личных данных пользователей. Он электронным способом привязывает ключ шифрования к информации о компании, что позволяет браузеру активировать “замок” и установить безопасное подключение к веб-серверу по протоколу HTTPS.
{: .prompt-tip }

Необходимо было заменить заканчивающийся ssl-сертификат для сайта, и в процессы замены из-за несоответствия фалов `example_ru.crt` и `example_ru.key` web-сервер apache не запускался.

Для проверки надо вычислить md5 сумму каждого файла, и эти суммы должны совпадать.

Переходим в каталог, где у нас лежат файлы

```sh
$ cd /etc/pki/tls/certs/
```

Вычисляем md5 сумму SSL-сертификата `example_ru.crt`

```sh
$ openssl x509 -noout -modulus -in example_ru.crt | openssl md5
(stdin)= 04c798ddffe50a19c7b1302b29e43798
```

Вычисляем md5 сумму приватного ключа `example_ru.key`

```sh
$ openssl rsa -noout -modulus -in example_ru.key | openssl md5
(stdin)= 04c798ddffe50a19c7b1302b29e43798
```

Вычисляем `md5` сумму модуля CSR

```sh
$ openssl req -noout -modulus -in example_ru.csr | openssl md5
(stdin)= 04c798ddffe50a19c7b1302b29e43798
```

Как видно, md5-суммы всех файлов совпадают
