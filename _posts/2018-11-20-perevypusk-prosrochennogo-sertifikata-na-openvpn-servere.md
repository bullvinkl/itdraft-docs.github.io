---
title: "Перевыпуск просроченного сертификата на OpenVPN-сервере"
date: "2018-11-20"
categories: 
  - Security-System
tags: 
  - "centos"
  - "openvpn"
image:
  path: /commons/0_HICLyAdNSIyT0ODU-1.jpg
  alt: "Перевыпуск просроченного сертификата на OpenVPN-сервере"
---

> **OpenVPN** - бесплатное программное решение для реализации подключений по протоколу VPN к виртуальным частным сетям, создания шифрованных туннелей между сервером и клиентскими компьютерами или подключения типа точка-точка для безопасной передачи данных через Интернет.
{: .prompt-tip }

При подключении к OpenVPN-серверу неожиданно стала появляться ошибка

```
Mon Nov 19 05:42:24 2018 VERIFY ERROR: depth=1, error=certificate has expired: C=RU, ST=ru, L=Moscow, O=Domain, CN=Domain CA, emailAddress=cert@example.com
Mon Nov 19 05:42:24 2018 OpenSSL: error:14090086:SSL routines:ssl3_get_server_certificate:certificate verify failed
Mon Nov 19 05:42:24 2018 TLS_ERROR: BIO read tls_read_plaintext error
Mon Nov 19 05:42:24 2018 TLS Error: TLS object -> incoming plaintext read error
Mon Nov 19 05:42:24 2018 TLS Error: TLS handshake failed
```

После анализа выяснялось, что срок действия сертификата центра сертификации (ca.crt) OpenVPN сервера истек.

Для устранения этой ошибки перевыпускаем самоподписанный сертификат центра сертификации

```sh
$ sudo openssl x509 -in ca.crt -days 3650 -out ca-new.crt -signkey ca.key
Getting Private key
```

где

- `ca.crt` - просроченный сертификат
- `ca-new.crt` - новый сертификат
- `ca.key` - ключ сертификата 
- `3650` - срок действия, в днях

Старый файл сертификата `ca.key` можно удалить, новый (`ca-new.crt`) переименовать в `ca.crt`

Проверяем

```sh
$ sudo openssl verify -CAfile ca.crt client-username.crt
client-username.crt: OK
```

Далее, перевыпускаем сертификат сервера

```sh
$ cd ../
$ sudo . ./vars
$ sudo ./build-key-server server
```

и перезапускаем OpenVPN

```sh
$ sudo service openvpn restart
```

Теперь, что бы пользователи смогли подключаться к OpenVPN-серверу надо что б они на ПК заменили старый  сертификат центра сертификации ca.crt на новый
