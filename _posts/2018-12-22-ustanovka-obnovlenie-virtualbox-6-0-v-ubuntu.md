---
title: "Установка - обновление Virtualbox 6.0 в Ubuntu"
date: "2018-12-22"
categories: 
  - Linux
tags: 
  - "ubuntu"
  - "virtualbox"
image:
  path: /commons/1353848_735f_3.jpg
  alt: "Установка - обновление Virtualbox 6.0 в Ubuntu"
---

> **VirtualBox** (Oracle VM VirtualBox) — программный продукт виртуализации для операционных систем Microsoft Windows, Linux, FreeBSD, macOS, Solaris/OpenSolaris, ReactOS, DOS и других.  
> Программа была создана компанией Innotek с использованием исходного кода Qemu. Первая публично доступная версия VirtualBox появилась 15 января 2007 года. В феврале 2008 года Innotek был приобретён компанией Sun Microsystems, модель распространения VirtualBox при этом не изменилась. В январе 2010 года Sun Microsystems была поглощена корпорацией Oracle, модель распространения осталась прежней.
{: .prompt-tip }

Открываем терминал и добавляем репозиторий Virtualbox

```sh
user@localhost~:$ sudo sh -c 'echo "deb [arch=amd64] http://download.virtualbox.org/virtualbox/debian $(lsb_release -sc) contrib" >> /etc/apt/sources.list.d/virtualbox.list'
```

Для Linux Mint $(lsb\_release -sc) заменить на bionic в ОС Mint 19.x, или на xenial в ОС Mint 18.x.  
Для Elementary 5.0 $(lsb\_release -sc) заменить на bionic.

Скачиваем и устанавливаем ключ для репозитория virtualbox

```sh
$ wget -q https://www.virtualbox.org/download/oracle_vbox_2016.asc -O- | sudo apt-key add -
```

Обновить кеш системных пакетов

```sh
$ sudo apt update
```

И наконец устанавливаем / обновляем VirtualBox 6.0

```sh
$ sudo apt install virtualbox-6.0
```

![](/assets/img/posts/2018/12/22/wp_virtualbox_6.png){: w="300" }

Для удаления VirtualBox 6.0 надо выполнить команду

```sh
$ sudo apt remove --autoremove virtualbox-6.0
```
