---
title: "Как установить Notepadqq (аналог Notepad++) в Ubuntu 18.04, 17.10"
date: "2018-08-16"
categories: 
  - Manuals
tags: 
  - "notepadqq"
  - "ubuntu"
image:
  path: /commons/mockup-2443050_1280.jpg
  alt: "Notepadqq в Ubuntu"
---

> **Notepadqq** - это текстовый редактор для операционной системы Linux, созданный как аналог Notepad++ для Windows. Он предназначен для программистов и разработчиков, предоставляя им функции, которые они обычно используют в таких редакторах как Notepad++.
{: .prompt-tip }

Рассмотрим варианты установки:

- установка Notepadqq из репозитория
- установка Notepadqq как snap пакета

## Установка Notepadqq из репозитория

Добавим репозиторий. Откройте терминал `Ctrl+Alt+T` и выполните команду

```sh
$ sudo add-apt-repository ppa:notepadqq-team/notepadqq
```

Синхронизируем индексные файловые пакеты с источниками

```sh
$ sudo apt-get update
```

Установим Notepadqq

```sh
$ sudo apt-get install notepadqq-gtk
```

## Удаление Notepadqq

Для удаления Notepadqq откройте терминал `Ctrl+Alt+T` и выполните команду

```sh
$ sudo apt-get remove --autoremove notepadqq-gtk
```

## Установка Notepadqq как snap пакета

Добавим поддержку snap в Ubuntu. Откройте терминал `Ctrl+Alt+T` и выполните команду

```sh
$ sudo apt-get install snapd snapd-xdg-open
```

Установим Notepadqq

```sh
$ snap install notepadqq
```
