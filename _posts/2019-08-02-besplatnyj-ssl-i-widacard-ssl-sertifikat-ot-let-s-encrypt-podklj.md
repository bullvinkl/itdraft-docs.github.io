---
title: "Бесплатный Widacard SSL от Let's Encrypt, подключение в Nginx и авто обновление на Centos 7"
date: "2019-08-02"
categories: 
  - Linux
  - Nginx
tags: 
  - "centos"
  - "lets-encrypt"
  - "nginx"
  - "ssl"
image:
  path: /commons/1037058_b070_2.jpg
  alt: "Widacard SSL от Let's Encrypt"
---

> **Wildcard-сертификат** — сертификат открытого ключа, который может использоваться с несколькими поддоменами *.example.ru
{: .prompt-tip }

Устанавливаем утилиту `certbot`

```sh
$ sudo yum install certbot
```

Получаем SSL сертификат. Тип проверки: TXT-запись в DNS

```sh
$ sudo certbot certonly --manual --agree-tos --email postmaster@example.ru --preferred-challenges=dns -d example.ru -d www.example.ru
```
- `certonly` — запрос нового сертификата;
- `manual` — проверка домена вручную.
- `preferred-challenges=dns` — метод проверки домена через dns.
- `agree-tos` — согласие на лицензионное соглашение;
- `email` — почтовый адрес администратора домена;
- `d` — перечисление доменов, для которых запрашиваем сертификат.

После чего `certbot` попросит для проверки прописать TXT-запись для доменных имен.  
Прописываем их на нашем dns-сервере, ждем некоторое время, чтоб они прописались и возвращаемся в консоль для подтверждения.

Обязательно надо выждать некоторое время, пока пропишется запись, иначе проверка не пройдет, вы не получите сертификат, и при следующем запросе txt-запись будет другой

Сервис Let's Encrypt так же выдает Wildcard SSL сертификаты. Для его получения выполним запрос:

```sh
$ sudo certbot certonly --manual --agree-tos --email postmaster@example.ru --preferred-challenges=dns -d example.ru -d *.example.ru
```

Срок действия SSL сертификата от Let's Encrypt ограничена `3 месяцами`, по-этому для обхода этого ограничения воспользуемся автоматическим обновлением

## Автоматическое продление SSL-сертификата

Ищем путь до certbot:

```sh
$ which certbot
/usr/bin/certbot
```

Запускаем редактирование cron и добавляем строку

```sh
$ crontab -e
0 0 * * 1,4 /usr/bin/certbot renew && systemctl restart nginx
```

В данном случае запуск скрипта проверки и продления сертификата (если срок действия сертификата заканчивается) будет происходить по понедельникам и четвергам в 00:00. После чего будет перезапущен Nginx

Остальные настройки рассматривались [ранее]({% post_url 2019-07-27-vkljuchaem-ssl-v-nginx-na-centos-7 %})

Часть конфига:

```
server {
        listen 443 ssl default_server;
        include snippets/ssl-params.conf;
        root /var/www/html;

        server_name example.ru www.example.ru;
        index index.php index.html index.htm;

        ssl_certificate /etc/letsencrypt/live/example.ru/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/example.ru/privkey.pem;
        ssl_trusted_certificate /etc/letsencrypt/live/example.ru/chain.pem;
		...
```
