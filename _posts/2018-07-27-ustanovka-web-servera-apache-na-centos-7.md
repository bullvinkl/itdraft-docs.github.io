---
title: "Установка Web-сервера Apache в Centos 7"
date: "2018-07-27"
categories: 
  - Linux
tags: 
  - "apache"
  - "centos"
image:
  path: /commons/mockup-2443050_1280.jpg
  alt: "Установка Web-сервера Apache"
---

> **Apache** HTTP-сервер — свободный веб-сервер. Apache является кроссплатформенным ПО, поддерживает операционные системы Linux, BSD, macOS, Microsoft Windows, Novell NetWare, BeOS. Основными достоинствами Apache считаются надёжность и гибкость конфигурации.
{: .prompt-tip }

Обновляем операционную систему

```sh
$ sudo yum update
```

Ставим Apache

```sh
$ sudo yum install httpd
```

Добавляем сервер в автозагрузку и запускаем его

```sh
$ sudo systemctl enable httpd
$ sudo systemctl start httpd
```

Открываем порты 80(http) и 443(https)

```sh
$ sudo firewall-cmd --permanent --zone=public --add-service=http
$ sudo firewall-cmd --permanent --zone=public --add-service=https
$ sudo firewall-cmd --reload
```
