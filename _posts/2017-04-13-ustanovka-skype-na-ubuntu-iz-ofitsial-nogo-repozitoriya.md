---
title: "Установка Skype на Ubuntu из официального репозитория"
date: "2017-04-13"
categories: 
  - Linux
tags: 
  - "skype"
  - "ubuntu"
image:
  path: /commons/156398bb67a3d4ee7ab97075a1137791.jpg
  alt: "Установка Skype на Ubuntu"
---

> Skype - это бесплатное проприетарное программное обеспечение с закрытым кодом, обеспечивающее текстовую, голосовую и видеосвязь через Интернет между компьютерами (IP-телефония), опционально используя технологии пиринговых сетей, а также платные услуги для звонков на мобильные и стационарные телефоны.
{: .prompt-tip }

Добавим репозиторий. Откройте терминал `Ctrl+Alt+T` и выполните команду

```sh
$ dpkg -s apt-transport-https > /dev/null || bash -c "sudo apt-get update; sudo apt-get install apt-transport-https -y"
$ curl https://repo.skype.com/data/SKYPE-GPG-KEY | sudo apt-key add -
$ echo "deb [arch=amd64] https://repo.skype.com/deb stable main" | sudo tee /etc/apt/sources.list.d/skype-stable.list
```

Синхронизируем индексные файловые пакеты с источниками

```sh
$ sudo apt-get update
```

Установим Skype

```sh
$ sudo apt-get install skypeforlinux -y
```
