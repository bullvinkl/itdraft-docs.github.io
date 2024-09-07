---
title: "Установка Kibana + Nginx Reverse Proxy в Centos 8"
date: "2020-11-10"
categories: 
  - Logging-System
  - DevOps
  - Web
tags: 
  - "centos"
  - "elk"
  - "kibana"
  - "nginx"
  - "reverse-proxy"
image:
  path: /commons/1039062_e553_4.jpg
  alt: "Установка Kibana"
---

> **Kibana** — это сервис для визуализации данных Elasticsearch и навигации их по Elastic Stack. Он помогает создавать дашборды, настраивать форму визуализации, формировать интерактивные графики, даже представлять геоданные, анализировать связи и изучать аномалии с машинным обучением.
{: .prompt-tip }

## Установка Kibana из репозитория

Импортируем PGP Key для дальнейшего добавления репозитория Elasticsearch

```sh
$ rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch
```

Добавляем репозиторий

```sh
$ sudo nano /etc/yum.repos.d/kibana.repo
[elasticsearch]
name=Elasticsearch repository for 7.x packages
baseurl=https://artifacts.elastic.co/packages/7.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
```

Устанавливаем Kibana

```sh
$ sudo dnf -y install kibana
```

Добавляем службу Kibana в автозагрузку и запускаем ее

```sh
$ sudo systemctl enable --now kibana
```

Проверяем, запустилась ли служба

```sh
$ systemctl status kibana
```

## Установка Kibana из RPM-пакета

Скачиваем пакет Kibana и устанавливаем его

```sh
$ wget https://artifacts.elastic.co/downloads/kibana/kibana-7.9.3-x86_64.rpm -P /tmp
$ cd /tmp
$ sudo dnf -y localinstall kibana-7.9.3-x86_64.rpm
```

Добавляем службу Kibana в автозагрузку и запускаем ее

```sh
$ sudo systemctl enable --now kibana
```

Проверяем, запустилась ли служба

```sh
$ sudo systemctl status kibana
```

Проверяем порт

```sh
$ netstat -tulnp | grep 5601
```

## Настройка Kibana

Редактируем конфигурационный файл Kibana

```sh
$ sudo nano /etc/kibana/kibana.yml
# Kibana is served by a back end server. This setting specifies the port to use.
server.port: 5601
# Specifies the address to which the Kibana server will bind. IP addresses and host names are both valid values.
# The default is 'localhost', which usually means remote machines will not be able to connect.
# To allow connections from remote users, set this parameter to a non-loopback address.
server.host: "192.168.1.10"
[…]

# The URLs of the Elasticsearch instances to use for all your queries.
elasticsearch.hosts: ["http://localhost:9200"]
```

Перезапускаем Kibana

```sh
$ sudo systemctl restart kibana
```

Если будем использовать Nginx as Reverse Proxy

```sh
...
server.host: "localhost"
...
```

в данном примере мы говорим, что сервер должен слушать на интерфейсе 192.168.1.10

## Настройка Firewall (если не используем Nginx as reverse proxy)

Открываем порт 5601

```sh
$ sudo firewall-cmd --add-port=5601/tcp --permanent
$ sudo firewall-cmd --reload
```

## Установка Nginx

Добавляем репозиторий NGINX

```sh
$ sudo nano /etc/yum.repos.d/nginx.repo
[nginx-stable]
name=nginx stable repo
baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=1
enabled=1
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true

[nginx-mainline]
name=nginx mainline repo
baseurl=http://nginx.org/packages/mainline/centos/$releasever/$basearch/
gpgcheck=1
enabled=0
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true
```

По умолчанию будет использоваться стабильная версия. Если нужна основная версия(mainline), переключаемся

```sh
$ sudo dnf config-manager --set-enabled nginx-mainline
```

Устанавливаем NGINX и httpd-tools

```sh
$ sudo dnf -y install nginx httpd-tools
```

Добавляем службу NGINX в автозагрузку и запускаем ее

```sh
$ sudo systemctl enable --now nginx
```

Проверяем, запустилась ли служба

```sh
$ systemctl status nginx
```

## Настройка Nginx

Отключаем дефолтный конфиг

```sh
$ cd /etc/nginx/conf.d/
$ sudo mv default.conf default.conf.disable
```

Создаем конфиг для Kibana

```sh
$ sudo nano kibana.conf
server {
  listen      80;
  #listen      [::]:80 ipv6only=on;
  server_name _;

  auth_basic "Restricted Access";
  auth_basic_user_file /etc/nginx/.kibana-user;

  location / {
    proxy_pass http://localhost:5601/;

    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
    proxy_cache_bypass $http_upgrade;
  }
}
```

Генерируем пароль для авторизации

```sh
$ sudo htpasswd -c /etc/nginx/.kibana-user mykibana
New password: password
Re-type new password: password
Adding password for user mykibana
```

Перезапускаем NGINX, смотрим статус

```sh
$ sudo systemctl reload nginx
$ systemctl status nginx
```

## Настройка Firewall

Открываем порт 80

```sh
$ sudo firewall-cmd --permanent --zone=public --add-service=http
$ sudo firewall-cmd --reload
```

Смотрим список правил

```sh
$ sudo firewall-cmd --list-all
```

Закрываем порт 5601, если ранее его открыывали

```sh
$ sudo firewall-cmd --permanent --zone=public --remove-port=5601/tcp
$ sudo firewall-cmd --reload
```

## Настройка SELinux

Добавляем правило в политику SELinux

```sh
$ sudo setsebool -P httpd_can_network_connect=1
```
