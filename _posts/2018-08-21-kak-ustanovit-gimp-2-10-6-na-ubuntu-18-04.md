---
title: "Как установить GIMP 2.10.6 на Ubuntu 18.04"
date: "2018-08-21"
categories: 
  - Linux
tags: 
  - "gimp"
  - "flatpak"
  - "ubuntu"
image:
  path: /commons/laptop-2.jpg
  alt: "Как установить GIMP на Ubuntu"
---

> **GIMP** (GNU Image Manipulation Program) — это мощный и функциональный инструмент для обработки растровой графики, который позволяет создавать и редактировать изображения, фотографии и дизайн-элементы. Он является альтернативой популярному программному обеспечению Adobe Photoshop, но является абсолютно бесплатным.
{: .prompt-tip }

Рассмотрим варианты установки:

- установка GIMP из репозитория
- установка GIMP как flatpak

## Установка GIMP из репозитория

Добавим репозиторий. Откройте терминал (Ctrl+Alt+T) и выполните команду

```sh
$ sudo add-apt-repository ppa:otto-kesselgulasch/gimp
```

Синхронизируем индексные файловые пакеты с источниками

```sh
$ sudo apt-get update
```

Установим GIMP

```sh
$ sudo apt-get install gimp
```

### Откат на предыдущую версию

Если по каким-либо причинам у вас не получается поставить последнюю версию GIMP, откройте терминал `Ctrl+Alt+T` и выполните команды

```sh
$ sudo apt-get install ppa-purge
$ sudo ppa-purge ppa:otto-kesselgulasch/gimp
```

## Установка GIMP как flatpak

Flatpak - это программная утилита для развертывания программного обеспечения, управления пакетами и виртуализации приложений для настольных компьютеров Linux.

Вначале необходимо добавить поддержку `flatpak` в Ubuntu, для этого откройте терминал `Ctrl+Alt+T` и выполните команды

```sh
$ sudo add-apt-repository ppa:alexlarsson/flatpak
$ sudo apt-get update
$ sudo apt-get install flatpak
```

Мы добавили репозиторий, синхронизировали индексные файловые пакеты с источниками и установили `flatpak`

Установим GIMP как `flatpak`, выполнив команду в терминале

```sh
$ flatpak install https://flathub.org/repo/appstream/org.gimp.GIMP.flatpakref
```

Что бы удалить GIMP

```sh
$ flatpak uninstall org.gimp.GIMP
```
