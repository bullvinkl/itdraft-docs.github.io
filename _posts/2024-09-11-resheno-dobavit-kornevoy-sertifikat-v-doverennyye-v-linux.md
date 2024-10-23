---
layout: post
title: "[Решено] Добавить корневой сертификат в доверенные в Linux"
date: "2024-09-11"
categories:
  - Manuals
  - Security-System
tags:
  - openssl
  - ubuntu
image:
  path: /commons/image5.webp
  alt: "Добавить корневой сертификат в доверенные в Linux"
---

> Корневой сертификат (CA) - это электронный документ, который подписывает центром сертификации SSL-сертификаты, выдаваемые для доменных имен. Корневой сертификат является частью ключа SSL и гарантирует, что выдавшая его организация верифицирована и легальна.
{: .prompt-tip }

## Для Debian-like дистрибутивов

Копируем наш корневой сертификат в формате `PEM` (сертификат начинается на `----BEGIN CERTIFICATE----`) в каталог `/usr/local/share/ca-certificates` и обновляем индекс сертификатов

```bash
$ sudo mv ./ca-itdraft.crt /usr/local/share/ca-certificates/
$ sudo update-ca-certificates
```

## Для RHEL-like дистрибутивов

Копируем наш корневой сертификат в каталог `/etc/pki/ca-trust/source/anchors` и обновляем индекс сертификатов

```bash
$ sudo mv ./ca-gge-ib.crt /etc/pki/ca-trust/source/anchors/
$ sudo update-ca-trust
```

## Проверка

Вывод команды:
```
Updating certificates in /etc/ssl/certs...
rehash: warning: skipping ca-certificates.crt,it does not contain exactly one certificate or CRL
1 added, 0 removed; done.
Running hooks in /etc/ca-certificates/update.d...
done.
```

- `1 added, 0 removed; done.` - сертификат добавился

Проверка:
```sh
$ openssl s_client -connect mail.itdraft.local:443 -CApath /etc/ssl/certs
```

Вывод команды:
```
CONNECTED(00000003)
...
---
Certificate chain
 0 s:C = RU, ST = Moscow, L = Moscow, O = ITDraft, OU = IT, CN = mail.itdraft.local
   i:DC = local, DC = itdraft, CN = Root CA
   a:PKEY: rsaEncryption, 2048 (bit); sigalg: RSA-SHA256
   v:NotBefore: Oct  8 10:22:21 2023 GMT; NotAfter: Oct  7 10:22:21 2025 GMT
---
Server certificate
-----BEGIN CERTIFICATE-----
...
-----END CERTIFICATE-----
...
SSL handshake has read 2505 bytes and written 439 bytes
Verification: OK
---
New, TLSv1.2, Cipher is ECDHE-RSA-AES256-GCM-SHA384
Server public key is 2048 bit
Secure Renegotiation IS supported
Compression: NONE
Expansion: NONE
No ALPN negotiated
SSL-Session:
    Protocol  : TLSv1.2
    Cipher    : ECDHE-RSA-AES256-GCM-SHA384
    Session-ID: 
    Session-ID-ctx: 
    Master-Key: ...
    PSK identity: None
    PSK identity hint: None
    SRP username: None
    Start Time: 1726034944
    Timeout   : 7200 (sec)
    Verify return code: 0 (ok)
    Extended master secret: no
---
...
```

В выводе смотрим:
- `Certificate chain` рядом с `i:`. Должен отображаться издатель: `i:DC = local, DC = itdraft, CN = Root CA`. Это говорит о том, что сервер представляет сертификат, подписанный нашим добавленным корневым сертификатом.
- `Verify return code`, значение должно быть `0 (ok)`.
