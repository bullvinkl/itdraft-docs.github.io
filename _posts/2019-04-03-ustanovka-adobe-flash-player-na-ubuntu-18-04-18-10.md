---
title: "Установка Adobe Flash Player в Ubuntu 18.04 / 18.10"
date: "2019-04-03"
categories: 
  - Manuals
tags: 
  - "flash"
  - "ubuntu"
image:
  path: /commons/987847_cbc9_2.jpg
  alt: "Adobe Flash Player в Ubuntu"
---

> **Adobe Flash Player** - облегченный плагин для браузеров, используемый для потоковой передачи видео, аудио и другого мультимедийного контента на сайтах и платформах Adobe Flash.  
> Пользователям веб-браузера Google Chrome не нужно устанавливать Adobe Flash Player, поскольку он поставляется с предустановленной собственной версией NPAPI.
{: .prompt-tip }

Добавляем репозиторий, обновляемся

```sh
$ sudo add-apt-repository "deb http://archive.canonical.com/ $(lsb_release -sc) partner"
$ sudo apt update
$ sudo apt -y upgrade
```

Устанавливаем Adobe Flash Plugin

```sh
$ sudo apt install adobe-flashplugin
```

Устанавливаем пакет rowser-plugin-freshplayer-pepperflash

```sh
$ sudo apt-get install browser-plugin-freshplayer-pepperflash
```
