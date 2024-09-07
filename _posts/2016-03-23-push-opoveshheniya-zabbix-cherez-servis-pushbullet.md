---
title: "Push-оповещения Zabbix через сервис Pushbullet"
date: "2016-03-23"
categories: 
  - Monitoring-System
tags: 
  - "push"
  - "zabbix"
image:
  path: /commons/hack_b-min-1.png
  alt: "Push-оповещения Zabbix через сервис Pushbullet"
---

> **Pushbullet** - это сервис для быстрой передачи файлов, ссылок, заметок и других данных между компьютером и мобильным устройством под управлением Android.
{: .prompt-tip }

## Подготовка

Регистрируемся в сервисе [Pushbullet](https://pushbullet.com) 

Получаем Token:
Settings - Account - Access Tokens

Скачиваем и устанавливаем приложение на телефон

## Подготовка скрипта

Создаем скрипт `/usr/lib/zabbix/alertscripts/pushbullet.sh`

```sh
$ sudo nano /usr/lib/zabbix/alertscripts/pushbullet.sh
#!/bin/bash
API_KEY="$1"
SUBJECT="$2"
MESSAGE="$3"
curl https://api.pushbullet.com/v2/pushes \
-u $1: \
-d type=note \
-d title="$SUBJECT" \
-d body="$MESSAGE" \
-X POST
```

где (данные параметры будут указываться в настройках Zabbix):  

- `$1` - наш Token
- `$2` - Тема
- `$3` - Сообщение

Делаем скрипт исполняемым

```sh
$ sudo chmod +x /usr/lib/zabbix/alertscripts/pushbullet.sh
```

## Настройка Zabbix

Переходим:
Администрирование - Способы оповещения

Нажимаем `Создать способ оповещения`

- Имя: `Pushbullet`
- Тип: `Скрипт`
- Имя скрипта: `pushbullet.sh` (полный путь указывать не надо)
- Параметры скрипта (появилось в Zabbix 3.0):
`Token`
`{ALERT.SUBJECT}`
`{ALERT.MESSAGE}`

Администрирование - Пользователи - выбираем пользователя - вкладка `Оповещение` и нажимаем `Добавить`

- Тип: `Pushbullet`
- Отправлять на: `указываем наш Token`
- Когда активен: `1-7,00:00-24:00 `(т.д. 7 дней в неделю, 24 часа в сутки)
- Использовать, если важность: указать, при какой важности отправлять сообщения (я обычно ставлю среднюю, высокую, черезвычайную)
- Активно: поставить галочку
