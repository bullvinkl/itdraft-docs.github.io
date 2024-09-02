---
title: "Получение и авто продление SSL-сертификата Let's Encrypt для Nginx в Centos 8 / Rocky Linux / AlmaLinux"
date: "2022-08-11"
categories: 
  - Linux
tags: 
  - "almalinux"
  - "centos"
  - "certbot"
  - "lets-encrypt"
  - "nginx"
  - "rocky-linux"
image:
  path: /commons/working-on-computer.jpg
  alt: "Получение и авто продление SSL-сертификата Let's Encrypt"
---

> **Certbot** - это клиент протокола ACME предназначенный для автоматического управления SSL-сертификатами от Let's Encrypt. Он позволяет полностью автоматизировать процесс получения и продления сертификата.

Устанавливаем утилиты

```sh
$ sudo dnf install certbot python3-certbot-nginx
```

Запускаем certbot для получения ssl-сертификата

```sh
$ sudo certbot --nginx -d itdraft.ru -d www.itdraft.ru
```

В процессе получения сертификата указываем наш e-mail, отвечаем на вопросы

Если сертификат успешно получен, в терминале увидим следующее

```
...
Successfully received certificate.
Certificate is saved at: /etc/letsencrypt/live/itdraft.ru/fullchain.pem
Key is saved at:         /etc/letsencrypt/live/itdraft.ru/privkey.pem
```

Создаем ключ dhparam

```sh
$ sudo openssl dhparam -out /etc/ssl/certs/dhparam.pem 2048
```

Добавляем скрипт перевыпуска ssl-сертификата в crontab

```sh
$ sudo crontab -e
# Раз в сутки, в 0:00
0 0 * * * /usr/bin/certbot renew > /dev/null 2>&1
```