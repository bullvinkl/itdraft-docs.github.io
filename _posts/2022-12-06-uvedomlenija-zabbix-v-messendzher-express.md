---
title: "Уведомления Zabbix в мессенджер eXpress"
date: "2022-12-06"
categories: 
  - Linux
  - Zabbix
tags: 
  - "express"
  - "zabbix"
image:
  path: /commons/pexels-christina-morillo-1181244-scaled.jpg
  alt: "Уведомления Zabbix в мессенджер eXpress"
---

> **eXpress** - это платформа корпоративных коммуникаций, которая сочетает в себе классический мессенджер, групповые аудио- и видеозвонки + единое окно корпоративных приложений Smart Apps для мобильного доступа ко всем информационным сервисам компании.

## Настройки в Express

Создаем бота через web-админку

```
NAME: Zabbix Бот    # Имя бота
APP_ID: zabbix_bot   # Идентификатор
URL: http://localhost/api/v1/zabbix_bot	   # Вставляем любой URL, т.к. поле обязательное
BOT_ID:	87016cb2-a373-543b-9336-237fc08873be      # Получаем ID
Секретный ключ:	968fd2a04ac500fb11f7b9a5986903f9
```

![](/assets/img/posts/2022/12/06/image-2.png){: w="300" }

Заходим в настройки бота и выставляем

```
allowed_data: none
```

Генерируем HMAC-SHA256 signature

```sh
$ echo -n <BOT_ID> | openssl dgst -sha256 -hmac <SECRET> | awk '{print toupper($0)}'

Пример:
$ echo -n 8111b2-a373-541-9116-211111e | openssl dgst -sha256 -hmac 9111111111115986903f9 | awk '{print toupper($0)}'
(STDIN)= E213F4CB1111111111344B04A78D90CC37FEF89339A57226DC
```

Получаем токен

```sh
$ curl 'https://%express_url%/api/v2/botx/bots/<BOT_ID>/token?signature=<SIGNATURE>'
```

Пример
```sh
$ curl 'https://%express_url%/api/v2/botx/bots/8111b2-a373-541-9116-211111e/token?signature=E213F4CB1111111111344B04A78D90CC37FEF89339A57226DC'
{"result":"sdfsdfsdfsdf.g2gDbsdfsdfsdfOTMzNi0yMzdmYzA4ODczYmVusdfsdfdsfAFRgA.-GsdfsdfsdfOwry-sdfsC-sdfsdfsdf8X4WAb4","status":"ok"}
```

## Настройки в Zabbix

Добавляем общий макрос

```
Администрирование > Общие > Макросы
Макрос: {$ZABBIX.URL}
Значение: URL zabbix, например http://192.168.1.90/
```

Добавляем способ оповещения (если нет)

```
Администрирование > Способы оповещений
Шаблон: https://git.zabbix.com/projects/ZBX/repos/zabbix/browse/templates/media/express.ms/media_express_ms.yaml
```

Задаем настройки, заходим в оповещение `Express.ms`

```
express_token: полученный токен # sdfsdfsdfsdf.g2gDbsdfsdfsdfOTMzNi0yMzdmYzA4ODczYmVusdfsdfdsfAFRgA.-GsdfsdfsdfOwry-sdfsC-sdfsdfsdf8X4WAb4
express_url: https://%url_нашего_корпоративного_express%
```

![](/assets/img/posts/2022/12/06/image-4.png){: w="300" }

## Настройки в Express - продолжение

Создаем новый чат или канал, в админке узнаем его `ID`

Добавляем нашего бота в этот чат / канал (с админскими правами, что бы мог публиковать сообщения)

## Настройки в Zabbix - продолжение

Переходим в настройки пользователя

```
Администрирование > Пользователи > наш пользователь
```

Добавляем в способы оповещения `Express.ms`

```
Отправлять на: ID канала (или ID пользователя) 49aa111e-ed83-5119-b9f2-11f1e3801b92 (узнали в предыдущем шаге)
```

![](/assets/img/posts/2022/12/06/image-5.png){: w="300" }