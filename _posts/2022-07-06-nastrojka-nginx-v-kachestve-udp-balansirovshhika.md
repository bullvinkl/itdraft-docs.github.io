---
title: "Настройка Nginx в качестве UDP-балансировщика"
date: "2022-07-06"
categories: 
  - Web
tags: 
  - "balancer"
  - "debian"
  - "nginx"
  - "udp"
image:
  path: /commons/computer.jpg
  alt: "Nginx в качестве UDP-балансировщика"
---

> В терминологии компьютерных сетей **балансировка нагрузки** или выравнивание нагрузки — метод распределения заданий между несколькими сетевыми устройствами с целью оптимизации использования ресурсов, сокращения времени обслуживания запросов, горизонтального масштабирования кластера, а также обеспечения отказоустойчивости.
{: .prompt-tip }

В одной из прошлых статей было рассмотрено как устанавливать Web-сервер Nginx в Debian или Centos

После установки, отключаем дефолтный конфиг

```sh
$ cd /etc/nginx/conf.d/
$ sudo mv default.conf default.conf.disable
```

Редактируем основной конфиг Nginx

```sh
$ sudo nano /etc/nginx/nginx.conf
...
stream {
  upstream backends {
    server 192.168.1.10:5060;
    server 192.168.1.11:5060;
  }
  server {
    listen 5060 udp;
    proxy_pass backends;
    proxy_responses 1;
  }
}

events {
    worker_connections  1024;
}
...
```

Проверяем конфиг на наличие ошибок

```sh
$ sudo nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

Перезапускаем Nginx

```sh
$ sudo systemctl restart nginx
```
