---
title: "Как установить Thunderbird 60 в Ubuntu 18.04"
date: "2018-09-19"
categories: 
  - Linux
tags: 
  - "thunderbird"
  - "ubuntu"
image:
  path: /commons/1032198_11a3_4.jpg
  alt: "установить Thunderbird в Ubuntu"
---

> **Thunderbird** - это бесплатное приложение для работы с электронной почтой, которое легко устанавливается и настраивается. Оно имеет множество характеристик и функций, включая фильтрацию и систематизацию писем, управление учетными записями и доступом к сообщениям, календарям и контактам.
{: .prompt-tip }

## Установка Thunderbird из репозитория

Добавим репозиторий. Откройте терминал (Ctrl+Alt+T) и выполните команду

```sh
$ sudo add-apt-repository ppa:mozillateam/thunderbird-next
```

Синхронизируем индексные файловые пакеты с источниками

```sh
$ sudo apt-get update
```

Установим Thunderbird

```sh
$ sudo apt-get install thunderbird
```

Если в системе до этого был установлен Thunderbird, то вместо предыдущей команды можно выполнить

```sh
$ sudo apt-get upgrade
```

## Удаление Thunderbird

Для удаления Notepadqq откройте терминал (Ctrl+Alt+T) и выполните команду

```sh
$ sudo apt-get remove --autoremove thunderbird
```
