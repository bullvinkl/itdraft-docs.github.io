---
title: "Получить SSL-сертификат Let’s Encrypt и подключить его в Apache для Centos 7"
date: "2018-06-19"
categories: 
  - Web
  - Automation
tags: 
  - "apache"
  - "centos"
  - "certbot"
  - "lets-encrypt"
  - "ssl"
image:
  path: /commons/1382848_75d8_2.jpg
  alt: "Получить SSL-сертификат Let’s Encrypt в Apache"
---

> **Let’s Encrypt** — это глобальный некоммерческий центр сертификации, который начал работу в 2014 году. Он предлагает бесплатные криптографические сертификаты для TLS-шифрования (HTTPS) доменов интернет-сайтов. Центр сертификации основан на принципах открытости, автоматизации и бесплатности, чтобы обеспечить безопасность и конфиденциальность интернет-трафика.
{: .prompt-tip }

Исходные данные:  

- На сервере уже установлен Apache
- Открыт ssl-порт (443)

Добавляем репозиторий EPEL и ставим mod-ssl

```sh
$ sudo yum install epel-release mod_ssl
```

Ставим `certbot` (клиента Let’s Encrypt)

```sh
$ sudo yum install python-certbot-apache
```

Получаем ssl-сертификат

```sh
$ sudo certbot --apache -d itdraft.ru -d www.itdraft.ru
```

В процессе установки будет запрошен e-mail, а затем скрипт спросит делать ли редирект в http на https

```sh
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Plugins selected: Authenticator apache, Installer apache
Enter email address (used for urgent renewal and security notices) (Enter 'c' to
cancel): admin@itdraft.ru
Starting new HTTPS connection (1): acme-v01.api.letsencrypt.org
-------------------------------------------------------------------------------
Please read the Terms of Service at
https://letsencrypt.org/documents/LE-SA-v1.2-November-15-2017.pdf. You must
agree in order to register with the ACME server at
https://acme-v01.api.letsencrypt.org/directory
-------------------------------------------------------------------------------
(A)gree/(C)ancel: A
-------------------------------------------------------------------------------
Would you be willing to share your email address with the Electronic Frontier
Foundation, a founding partner of the Let's Encrypt project and the non-profit
organization that develops Certbot? We'd like to send you email about EFF and
our work to encrypt the web, protect its users and defend digital rights.
-------------------------------------------------------------------------------
(Y)es/(N)o: Y
Please choose whether or not to redirect HTTP traffic to HTTPS, removing HTTP access.
-------------------------------------------------------------------------------
1: No redirect - Make no further changes to the webserver configuration.
2: Redirect - Make all requests redirect to secure HTTPS access. Choose this for
new sites, or if you're confident your site works on HTTPS. You can undo this
change by editing your web server's configuration.
-------------------------------------------------------------------------------
Select the appropriate number [1-2] then [enter] (press 'c' to cancel): 2
```

После завершения установки будет следующее сообщение:

```sh
IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/itdraft.ru/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/itdraft.ru/privkey.pem
   Your cert will expire on 2018-09-17. To obtain a new or tweaked
   version of this certificate in the future, simply run certbot again
   with the "certonly" option. To non-interactively renew *all* of
   your certificates, run "certbot renew"
 - If you like Certbot, please consider supporting our work by:
   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le
```

## Автообновление ssl-сертификата

Редактируем crontab:

```sh
$ sudo crontab -e
```

Добавляем строчку:

```sh
0 0 * * 1 /usr/bin/certbot renew >> /var/log/sslrenew.log
```

Либо запускаем вручную:

```sh
$ sudo certbot renew
Saving debug log to /var/log/letsencrypt/letsencrypt.log
-------------------------------------------------------------------------------
Processing /etc/letsencrypt/renewal/itdraft.ru.conf
-------------------------------------------------------------------------------
Cert not yet due for renewal
-------------------------------------------------------------------------------
The following certs are not due for renewal yet:
  /etc/letsencrypt/live/itdraft.ru/fullchain.pem expires on 2018-09-17 (skipped)
No renewals were attempted.
-------------------------------------------------------------------------------
```
