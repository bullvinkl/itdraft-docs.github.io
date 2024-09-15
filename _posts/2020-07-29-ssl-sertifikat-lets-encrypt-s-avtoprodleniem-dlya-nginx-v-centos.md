---
title: "SSL-сертификат Let’s Encrypt с авто продлением для Nginx в Centos"
date: "2020-07-29"
categories: 
  - Web
  - Automation
tags: 
  - "centos"
  - "lets-encrypt"
  - "nginx"
  - "ssl"
  - "crontab"
image:
  path: /commons/1032198_11a3_4.jpg
  alt: "Let’s Encrypt"
---

> **Let’s Encrypt** — центр сертификации, начавший работу в бета-режиме с 3 декабря 2015 года, предоставляющий бесплатные криптографические сертификаты X.509 для TLS-шифрования. Процесс выдачи сертификатов полностью автоматизирован.
{: .prompt-tip }

## Подготовка

Устанавливаем пакеты

```sh
$ sudo yum -y install git bc
```

Клонируем GitHub репозиторий `letsencrypt` в каталог `/opt/letsencrypt`

```sh
$ sudo git clone https://github.com/letsencrypt/letsencrypt /opt/letsencrypt
```

## Получаем ssl-сертификат

Переходим в каталог

```sh
$ cd /opt/letsencrypt
```

Запускаем скрипт

```sh
$ ./letsencrypt-auto certonly -a webroot --webroot-path=/var/www/example.com/public_html -d example.com -d www.example.com
```

> - `example.com, www.example.com` — наш домен
> - `webroot-path=/var/www/example.com/public_html` - директория, где расположен сайт
{: .prompt-info }

> Запускаем приложение letsencrypt-auto без `sudo`
{: .prompt-warning }

Если скрипт отработал успешно, мы получим сообщение:

```
IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/www.example.com/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/www.example.com/privkey.pem
   Your cert will expire on 2020-10-26. To obtain a new or tweaked
   version of this certificate in the future, simply run
   letsencrypt-auto again. To non-interactively renew *all* of your
   certificates, run "letsencrypt-auto renew"
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le
```

В результате в каталоге `/etc/letsencrypt/live/www.example.com/` мы получили следующие файлы

> - `cert.pem` - сертификат
> - `chain.pem` - цепь сертификатов Let’s Encrypt
> - `fullchain.pem` - объединенные сертификаты cert.pem и chain.pem
> - `privkey.pem` - приватный ключ
{: .prompt-info }

Сгенерируем ключ Диффи-Хеллмана и сохраним его в каталог `/etc/ssl/certs/`

```sh
$ sudo openssl dhparam -out /etc/ssl/certs/dhparam.pem 2048
```

## Настройка Nginx

В итоге наш конфиг будет выглядеть так

```sh
$ sudo nano /etc/nginx/sites-available/example.com.conf

server {
    # redirect
    server_name example.com www.example.com
    listen 80;
    return 301 https://www.example.com$request_uri;
}

server {
        listen 443 ssl;

        server_name example.com www.example.com;

        # Let's Encrypt certs
        ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem; 
        ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
		ssl_trusted_certificate /etc/letsencrypt/live/www.itdraft.ru/chain.pem;

        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        ssl_prefer_server_ciphers on;

        ssl_dhparam /etc/ssl/certs/dhparam.pem;
        ssl_ciphers 'ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-DSS-AES128-GCM-SHA256:kEDH+AESGCM:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA:ECDHE-ECDSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-DSS-AES128-SHA256:DHE-RSA-AES256-SHA256:DHE-DSS-AES256-SHA:DHE-RSA-AES256-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:AES:CAMELLIA:DES-CBC3-SHA:!aNULL:!eNULL:!EXPORT:!DES:!RC4:!MD5:!PSK:!aECDH:!EDH-DSS-DES-CBC3-SHA:!EDH-RSA-DES-CBC3-SHA:!KRB5-DES-CBC3-SHA';
        ssl_session_timeout 1d;
        ssl_session_cache shared:SSL:50m;

        ssl_stapling on;
        ssl_stapling_verify on;
        add_header Strict-Transport-Security max-age=15768000;

        location ~ /.well-known {
                allow all;
        }

        # The rest of your server block
        root /usr/share/nginx/html;
        index index.html index.htm;

        location / {
                # First attempt to serve request as file, then
                # as directory, then fall back to displaying a 404.
                try_files $uri $uri/ =404;
                # Uncomment to enable naxsi on this location
                # include /etc/nginx/naxsi.rules
        }
}
```

Проверяем и перезагружаем Nginx

```sh
$ sudo nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful

$ sudo systemctl reload nginx
```

## Авто продление SSL-сертификата

Для авто продления используется команда

```sh
$ /opt/letsencrypt/letsencrypt-auto renew
```

Т.к. мы недавно обновляли сертификат, то в ответ увидим сообщение

```sh
Requesting to rerun /opt/letsencrypt/letsencrypt-auto with root privileges...
Saving debug log to /var/log/letsencrypt/letsencrypt.log

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Processing /etc/letsencrypt/renewal/www.example.com.conf
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Cert not yet due for renewal

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -

The following certs are not due for renewal yet:
  /etc/letsencrypt/live/www.example.com/fullchain.pem expires on 2020-10-26 (skipped)
No renewals were attempted.
```

Добавим команду для авто продления в `crontab`

```sh
$ sudo crontab -e
## Обновление SSL сертификата по понедельникам и четвергам в 2:15
15  2  * * 1,4  /opt/letsencrypt/letsencrypt-auto renew && systemctl restart nginx
```

## Обновление скрипта letsencrypt

Скачиваем изменения для GitHub репозитория `letsencrypt`

```sh
$ cd /opt/letsencrypt
$ sudo git pull
```
