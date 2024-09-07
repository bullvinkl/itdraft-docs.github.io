---
title: "Установка ImageMagick для PHP в CentOS 7"
date: "2019-08-01"
categories: 
  - Linux
tags: 
  - "centos"
  - "imagemagic"
  - "php"
image:
  path: /commons/p9lahdjjxckbx3hsm3uqaioj4bu.png
  alt: "Установка ImageMagick для PHP"
---

> **ImageMagick** — набор программ (консольных утилит) для чтения и редактирования файлов множества графических форматов. Является свободным и кроссплатформенным программным обеспечением.
{: .prompt-tip }

Установим необходимые пакеты

```sh
$ sudo yum install gcc php-devel php-pear
```

Запускаем установку ImageMagick

```sh
$ sudo yum install ImageMagick ImageMagick-devel
```

Устанавливаем php-модуль для функционирования ImageMagick в php скриптах, и активируем его

```sh
$ sudo pecl install imagick
$ echo "extension=imagick.so" | sudo tee -a /etc/php.d/imagick.ini
```

Теперь осталось перезапустить либо web-сервер apache, либо php-fpm если вы использует связку nginx + php-fpm

```sh
$ sudo systemctl restart apache
$ sudo systemctl restart php-fpm
```
