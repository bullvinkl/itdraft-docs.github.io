---
title: "YUM и WGET через прокси"
date: "2015-07-29"
categories: 
  - Manuals
tags: 
  - "centos"
  - "linux"
  - "proxy"
  - "wget"
  - "yum"
image:
  path: /commons/transformers-min.png
  alt: "YUM и WGET через прокси"
---

> **YUM** (Yellowdog Updater, Modified) - пакетный менеджер для Linux-дистрибутивов, таких как RedHat, CentOS, Scientific Linux и другие. Он позволяет управлять установкой, обновлением и удалением пакетов программного обеспечения.
> **WGET** - утилита для загрузки файлов с веб-сайтов. Она позволяет скачивать файлы рекурсивно, обрабатывать ссылки и продолжать загрузку с того места, где была прервана.
{: .prompt-tip }

## WGET через прокси

Редактируем файл `/etc/wgetrc`

```sh
$ sudo nano /etc/wgetrc
http_proxy = http://192.168.0.1:3128
ftp_proxy = http://192.168.0.1:3128
use_proxy = on
proxy-user = user
proxy-password = password
```
  
## YUM через прокси

Редактируем файл `/etc/yum.conf`

```sh
$ echo proxy=http://user:password@proxy:port/ | sudo tee -a /etc/yum.conf
$ sudo yum update
```

## RPM Package Manager через прокси

Для RPM Package Manager задать переменные среды в файле `/etc/profile` или `~/.bash_profile`

```sh
$ sudo export http_proxy=http://user:password@proxy:port/  
$ sudo export ftp_proxy=http://user:password@proxy:port/
```
