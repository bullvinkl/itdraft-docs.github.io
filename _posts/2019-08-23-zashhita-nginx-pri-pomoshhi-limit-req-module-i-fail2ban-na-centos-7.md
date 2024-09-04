---
title: "Защита Nginx при помощи Limit Req Module и Fail2Ban на Centos 7"
date: "2019-08-23"
categories: 
  - Linux
  - Nginx
  - Fail2Ban
tags: 
  - "fail2ban"
  - "limit_req"
  - "nginx"
image:
  path: /commons/0_HICLyAdNSIyT0ODU-1.jpg
  alt: "Защита Nginx"
---

> Модуль **ngx_http_limit_req_module** позволяет ограничить скорость обработки запросов по заданному ключу или, как частный случай, скорость обработки запросов, поступающих с одного IP-адреса.

## Принцип действия

Модуль Nginx’s Limit Req Module ограничивает максимальное количество запросов с одного IP и записывает информацию в журнал error.log  
Fail2Ban считывает данные из журнала error.log и блокирует IP, который превысил лимит запросов.

## Настройка NGINX

[Установка Nginx]({% post_url 2019-07-15-ustanovka-web-servera-nginx-dlya-raboty-s-virtualnymi-hostami-php-fpm-v-rezhime-raboty-sock-mysql-server-mariadb-na-centos-7 %}) была рассмотрена ранее

Вначале настроим Nginx на ограничения количества запросов c одного IP-адреса с помощью модуля `Nginx’s Limit Req Module`.

Открываем файл `/etc/nginx/nginx.conf` (для всех сайтов на сервере) и в блок `http {...}` добавляем строку:

```
http {
...
limit_req_zone $binary_remote_addr zone=one:10m rate=1r/s;
...
}
```

- `$binary_remote_addr` - переменная Nginx содержащая IP с которого пришёл запрос;
- `zone=one` - имя зоны. Если настройка делается в нескольких vhost файлах, имена должны быть разные;
- `10m` - размер зоны. 1m может содержать 16000 состояний, т.е. 16000 уникальных IP-адресов;
- `rate=1r/s` - разрешено 1 запрос в секунду. Так же можно указать количество запросов в минуту (30r/m — 30 запросов в минуту)

Если вы используете виртуальные хосты, строчку `limit_req_zone` надо прописать за пределами блока `server {...}`

Теперь для нашего хоста в блок `server {...}` добавим строку:

```
limit_req   zone=one  burst=1 nodelay;
```

- `one` - имя настроенной выше зоны;
- `burst` - максимальный всплеск активности, можно регулировать до какого значения запросов в секунду может быть всплеск запросов;
- `nodelay` - незамедлительно, при достижении лимита подключений, выдавать код 503 (Service Unavailable) для этого IP

Пример конфига для запросов к php файлам:

```
...
location ~ \.php$ {
  limit_req   zone=one  burst=1 nodelay;
  include fastcgi_params;
  fastcgi_pass unix:/var/run/php-fpm/php-fpm.sock;
}
...
```

## Настройка Fail2Ban

[Установка Fail2Ban]({% post_url 2019-04-10-ustanovka-i-nastrojka-fail2ban-v-centos-7 %}) была рассмотрена ранее

Отредактируем файл `/etc/fail2ban/jail.local`, добавим строки в самом конце файла:

```
[nginx-limit-req]
enabled = true
filter  = nginx-limit-req
port    = http,https
logpath = /var/log/nginx/*error.log
bantime = 600
maxretry = 5
```

- `enabled` - включить или выключить правило
- `logpath` - путь к логам, если используете виртуальные хосты и логи хранятся в другом месте, надо прописать соответствующий путь
- `filter` - название фильтра (берется из каталога `/etc/fail2ban/filter.d/`)
- `maxretry` - количество записей в логе об ошибке для одного IP
- `bantime` - время блокировки ip, в секундах

Перезагружаем Fail2Ban:

```sh
$ sudo systemctl restart fail2ban
```

или перечитываем файлы конфигурации

```sh
$ sudo fail2ban-client reload
```

Посмотреть информацию о заблокированных IP

```sh
$ sudo fail2ban-client status nginx-limit-req
```

## Другие правила для Fail2Ban

### Блокируем ботов

Переходим в катало `/etc/fail2ban/filter.d/` и копируем фильтр для Apache

```sh
$ sudo cd /etc/fail2ban/filter.d/
$ sudo cp apache-badbots.conf nginx-badbots.conf
```

Редактируем файл `/etc/fail2ban/jail.local`, дописываем строки:

```
[nginx-badbots]
enabled  = true
filter   = nginx-badbots
port     = http,https
logpath  = /var/log/nginx/access.log
bantime  = 600
maxretry = 2
```

Данный фильтр банит по `userugent`

Не забываем указать корректный путь для `logpath`

### Блокируем IP, которые обращаются к скриптам

Переходим в катало `/etc/fail2ban/filter.d/` и создаем файл `nginx-noscript.conf`

```sh
$ sudo nano /etc/fail2ban/filter.d/nginx-noscript.conf
[Definition]
failregex = ^<HOST> -.*GET.*(\.php|\.asp|\.exe|\.pl|\.cgi|\.scgi)
ignoreregex =
```

Далее, редактируем файл `/etc/fail2ban/jail.local`, дописываем строки:

```
[nginx-noscript]
enabled  = true
filter   = nginx-noscript
port     = http,https
logpath  = /var/log/nginx/access.log
bantime  = 600
maxretry = 6
```

Фильтр блокирует ip, которые напрямую обращаются к скриптам с расширением php, asp, exe, pl, cgi, scgi

### Фильтр noproxy

Переходим в катало `/etc/fail2ban/filter.d/` и создаем файл `nginx-noproxy.conf`

```sh
$ sudo nano /etc/fail2ban/filter.d/nginx-noproxy.conf
[Definition]
failregex = ^<HOST> -.*GET http.*
ignoreregex =
```

Далее, редактируем файл `/etc/fail2ban/jail.local`, дописываем строки:

```
[nginx-noproxy]
enabled  = true
filter   = nginx-noproxy
port     = http,https
logpath  = /var/log/nginx/access.log
bantime  = 600
maxretry = 2
```

### Фильтр connect

Переходим в катало `/etc/fail2ban/filter.d/` и создаем файл `nginx-noconnect.conf`

```sh
$ sudo nano /etc/fail2ban/filter.d/nginx-noconnect.conf
[Definition]
failregex = ^<HOST> -.*CONNECT
ignoreregex =
```

Далее, редактируем файл `/etc/fail2ban/jail.local`, дописываем строки:

```
[nginx-noconnect]
enabled  = true
filter   = nginx-noconnect
port     = http,https
logpath  = /var/log/nginx/access.log
bantime  = 600
maxretry = 2
```

Фильтр блокирует IP, которые используют метод Connect

После добавления фильтров перезагружаем `fail2ban` и проверяем статус

```sh
$ sudo fail2ban-client restart
$ sudo fail2ban-client status
```

Что бы посмотреть информацию по заблокированным ip по фильтрам используем команду

```sh
$ sudo fail2ban-client status %filter%
```

где `%filter% `- название фильтра

Например для фильтра `nginx-badbots` команда будет выглядеть:

```sh
$ sudo fail2ban-client status nginx-badbots
```

Чтобы разбанить IP можно воспользоваться командой

```sh
$ sudo fail2ban-client set nginx-badbots unbanip 8.8.8.8
```