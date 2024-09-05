---
title: "Установка / обновление Firefox 62 на Ubuntu 18.04"
date: "2018-09-06"
categories: 
  - Linux
tags: 
  - "firefox"
  - "ubuntu"
image:
  path: /commons/c629418_a5b0-1.jpg
  alt: "Установка / обновление Firefox"
---

> **Firefox** - это бесплатный веб-браузер, разработанный Mozilla, некоммерческой организацией. Он доступен на русском языке и может быть загружен на компьютеры, мобильные устройства или организационные системы.

## Установка Firefox из репозитория

Добавим репозиторий. Откройте терминал (Ctrl+Alt+T) и выполните команду

```sh
$ sudo add-apt-repository ppa:mozillateam/firefox-next
```

Синхронизируем индексные файловые пакеты с источниками

```sh
$ sudo apt-get update
```

Установим Firefox

```sh
$ sudo apt-get install -y firefox
```

Если Firefox уже был установлен в системе, то можно выполнить команду:

```sh
$ sudo apt-get update && sudo apt-get upgrade -y
```

что бы обновить все пакеты в системе.

Либо выполнить команду, что бы обновить только firefox

```sh
$ sudo apt-get install -y --only-upgrade firefox
```