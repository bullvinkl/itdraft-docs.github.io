---
title: "Установка Nginx в Debian 11 из репозитория"
date: "2022-07-04"
categories: 
  - Web
tags: 
  - "debian"
  - "nginx"
image:
  path: /commons/computer2.jpg
  alt: "Установка Nginx"
---

> **Nginx** — веб-сервер и почтовый прокси-сервер, работающий на Unix-подобных операционных системах. Начиная с версии 0.7.52 появилась экспериментальная бинарная сборка под Microsoft Windows.
{: .prompt-tip }

Загружаем ключ для подписи Nginx

```sh
$ sudo wget https://nginx.org/keys/nginx_signing.key
```

Устанавливаем утилиту gnupg2 что бы в дальнейшем добавить скаченный ключ

```sh
$ sudo apt update
$ sudo apt install -y gnupg2
```

Добавляем загруженный ключ в список программных ключей

```sh
$ sudo apt-key add nginx_signing.key
```

Добавляем репозиторий Nginx

```
$ sudo nano /etc/apt/sources.list
...
# NGINX repo
deb https://nginx.org/packages/mainline/debian/ bullseye nginx
deb-src https://nginx.org/packages/mainline/debian bullseye nginx
```

Устанавливаем Nginx

```sh
$ sudo apt update
$ sudo apt install -y nginx
```

Запускаем Nginx и добавляем его в автозагрузку

```sh
$ sudo systemctl start nginx
$ sudo systemctl enable nginx
```
