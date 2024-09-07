---
title: "Установка sendmail в CentOS и отправка писем из консоли"
date: "2015-07-21"
categories: 
  - Manuals
tags: 
  - "centos"
  - "sendmail"
image:
  path: /commons/0_HICLyAdNSIyT0ODU-1.jpg
  alt: "Установка sendmail в CentOS"
---

> **sendmail** - это программный продукт, предназначенный для доставки электронной почты (email). Он работает как агент маршрутизации сообщений, обеспечивая передачу почты между почтовыми серверами и клиентскими приложениями.
{: .prompt-tip }

Ставим sendmail

```sh
$ sudo yum install sendmail sendmail-cf -y
```

Добавляем службу в автозагрузку

```sh
$ sudo chkconfig --level 345 sendmail on
```

Запускаем службу

```sh
$ sudo service sendmail start
tarting sendmail: [  OK  ]
Starting sm-client: [  OK  ]
```

Тестируем (вместо `user@user.com` вставьте ваш e-mail):

```sh
$ sudo echo "Test body" | mail -s "Test subject" user@user.com
```

Вероятно письмо попадет в папку СПАМ.
