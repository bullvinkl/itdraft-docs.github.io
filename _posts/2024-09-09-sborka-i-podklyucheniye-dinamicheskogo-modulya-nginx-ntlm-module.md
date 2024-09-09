---
layout: post
title: "Сборка и подключение динамического модуля nginx-ntlm-module"
date: "2024-09-09"
categories:
  - Web
tags:
  - nginx
  - ubuntu
image:
  path: /commons/964732_881f.jpg
  alt: "Сборка и подключение динамического модуля nginx-ntlm-module"
---

> Модуль **nginx-ntlm-module** - это дополнительный модуль для сервера web-приложений nginx, позволяющий прозрачно проксировать запросы с использованием механизма аутентификации NTLM.
> Модуль создан для обеспечения прозрачного проксирования запросов с клиентами, использующими NTLM для аутентификации. Он создает связь с upstream-сервером только после получения запроса с заголовком “Authorization”, начинавшимся с “Negotiate” или “NTLM”. Далее все последующие запросы клиента будут направляться через эту связь, сохраняя контекст аутентификации.
{: .prompt-tip }

Смотрим версию Nginx, который установлен

```sh
$ nginx -v
nginx version: nginx/1.18.0 (Ubuntu)
```

Качаем исходники, распаковываем, переходим в каталог

```sh
$ wget https://nginx.org/download/nginx-1.18.0.tar.gz
$ tar -zxf nginx-*.tar.gz
$ cd nginx-*/
```

Клонируем репозиторий `nginx-ntlm-module`

```sh
$ git clone https://github.com/gabihodoroaga/nginx-ntlm-module.git
```

Конфигурируем и собираем модуль

```sh
$ ./configure --with-compat --add-dynamic-module=nginx-ntlm-module
$ make
$ make modules
```

В каталоге `objs` появится файл `ngx_http_upstream_ntlm_module.so`

Копируем его в каталог со всеми мудулями

```sh
$ sudo cp objs/ngx_http_upstream_ntlm_module.so /usr/lib/nginx/modules/
```

Подключаем модуль, для этого создаем конфигурационный файл

```sh
$ sudo nano /etc/nginx/modules-enabled/nginx-ntlm-module.conf
load_module modules/ngx_http_upstream_ntlm_module.so;
```

Проверяем, если ошибок нет, перезагружаем Nginx

```sh
$ sudo nginx -t
$ sudo systemctl restart nginx
```
