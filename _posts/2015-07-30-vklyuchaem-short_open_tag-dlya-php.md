---
title: "Включаем short_open_tag для php"
date: "2015-07-30"
categories: 
  - Web
tags: 
  - "php"
  - "short_open_tag"
image:
  path: /commons/computer.jpg
  alt: "short_open_tag"
---

> **short_open_tag** в PHP - это директива, определяющая, будет ли обрабатываться PHP-код, написанный между тегами <? и ?>. Это позволяет писать PHP-код в традиционном стиле, используя короткие теги, вместо обязательного использования <?php и ?>.
{: .prompt-tip }

Редактируем файл php.ini:  

```sh
$ sudo nano /etc/php.ini
```

Правим строчку `short_open_tag = Off` на `short_open_tag = On`
  
Перезапускаем апач:  

```sh
$ sudo /etc/ini.t/httpd restart
```
