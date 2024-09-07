---
layout: post
title: "Скрипт проверки срока действия SSL-сертификата"
date: "2024-09-07"
categories:
  - Automation
tags:
  - linux
  - bash
  - crontab
  - telegram
image:
  path: /commons/virtualisationimage.png
  alt: "Скрипт проверки срока действия SSL-сертификата"
---

> **Срок действия SSL-сертификата** - это ограниченный период времени, в течение которого SSL-сертификат остается действительным и обеспечивает безопасное шифрование данных между браузером и веб-сервером. Срок действия обычно указывается в днях, месяцах или годах и варьируется в зависимости от типа сертификата и сертификационного центра, который его выдал.
{: .prompt-tip }

Скрипт проверки срока действия SSL-сертификата и отправки уведомления в Telegram:

```sh
$ cat ~/check_ssl.sh
#!/bin/bash

# Telegram
TOKEN="???" #token
ID="???" #id chanel
URL="https://api.telegram.org/bot$TOKEN/sendMessage"
URLDOC="https://api.telegram.org/bot$TOKEN/sendDocument"

# Variables
SERVER=$1
DAYS=30

# Calculate & Send
EXPIRE_DATE=`echo | openssl s_client -connect $SERVER:443 -servername $SERVER -tlsextdebug 2>/dev/null | openssl x509 -noout -dates 2>/dev/null | grep notAfter | cut -d'=' -f2`
EXPIRE_SECS=`date -d "${EXPIRE_DATE}" +%s`
EXPIRE_TIME=$(( ${EXPIRE_SECS} - `date +%s` ))
if test $EXPIRE_TIME -lt 0; then
  RETVAL=0
else
  RETVAL=$(( ${EXPIRE_TIME} / 24 / 3600 ))
fi
if [ "${RETVAL}" -le $DAYS ]; then
  curl -s -X POST $URL \
    -d parse_mode=HTML \
    -d chat_id=$ID \
    -d text="<b>$SERVER:</b>%0ASSL-сертификат закончится через: ${RETVAL} дня/дней"
fi
```

где
`DAYS=30` - когда начинать отправлять уведомления

Делаем скрипт испольняемым

```sh
$ sudo chmod +x ~/check_ssl.sh
```

Добавляем его в cron

```sh
$ crontab -e
## Проверка валидности сертификатов в 2:15 утра
15 02 * * * ~/check_ssl.sh itdraft.ru
```
