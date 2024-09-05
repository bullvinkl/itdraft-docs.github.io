---
title: "Как установить LibreOffice 6.1 в Ubuntu 18.04, 16.04"
date: "2018-08-16"
categories: 
  - Linux
tags: 
  - "libreoffice"
  - "ubuntu"
image:
  path: /commons/virtual-reality-development.jpg
  alt: "установить LibreOffice"
---

> **LibreOffice** - мощный и бесплатный офисный пакет, который обеспечивает полную совместимость с 32/64-битными системами. Он переведен на более чем 30 языков мира, включая русский. Поддерживает большинство популярных операционных систем, включая GNU/Linux, Microsoft Windows и Mac OS X.

Рассмотрим варианты установки

- установить LibreOffice 6.1 как snap пакет
- установить LibreOffice 6.1 из репозитория

## Для установки LibreOffice 6.1 как snap пакета

Откройте терминал (Ctrl+Alt+T) и выполните команду

```sh
$ snap install libreoffice
```

## Для установки LibreOffice 6.1 из репозитория

Добавим репозиторий, откройте терминал (Ctrl+Alt+T) и выполните команду

```sh
$ sudo add-apt-repository ppa:libreoffice/ppa
```

Синхронизируем индексные файловые пакеты с источниками

```sh
$ sudo apt-get update
```

Установим LibreOffice 6.1

```sh
$ sudo apt-get install libreoffice
```

Чтобы откатиться до предыдущей версии, в терминале выполните команду

```sh
$ sudo apt-get install ppa-purge
$ sudo ppa-purge ppa:libreoffice/ppa
```