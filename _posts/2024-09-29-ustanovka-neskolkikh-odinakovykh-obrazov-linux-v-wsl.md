---
layout: post
title: "Установка нескольких одинаковых образов Linux в WSL"
date: "2024-09-29"
categories:
  - Development
tags:
  - wsl
  - powershell
  - ubuntu
image:
  path: /commons/Windows-10-Uzak.webp
  alt: "Установка нескольких одинаковых образов Linux в WSL"
---

> **WSL** (Windows Subsystem for Linux) - это функция операционной системы Windows, которая позволяет запускать среду Linux на компьютере Windows без необходимости в отдельной виртуальной машине или двойной загрузке.
{: .prompt-tip }

## Первый способ. Используя образ из интернета

Создаем проект, куда будет скачен образ, и куда будут установлены дистрибутивы

```sh
PS> cd
PS> mkdir wsl
PS> cd wsl
```

Скачиваем образ

```sh
PS> curl (("https://cloud-images.ubuntu.com",
"releases/24.04/release/",
"ubuntu-24.04-server-cloudimg-amd64-root.tar.xz") -join "/") `
--output ubuntu-24.04-wsl-root-tar.gz
```

Для установки дистрибутивов, будем использовать следующую команду

```sh
PS> wsl --import <Distribution Name> <Installation Folder> <Ubuntu WSL2 Image Tarball path>
```

где:
> - <Distribution Name> - имя дистрибутива
> - <Installation Folder> - каталог, куда будет установлен дистрибутив
> - <Ubuntu Tarball path> - путь к tar-архиву образа Ubuntu WSL2, который мы скачали ранее.
{: .prompt-info }

Создаем каталоги, куда будем устанавливать дистрибутивы

```sh
PS> mkdir Ubuntu-1,Ubuntu-2
```

Устанавливаем первый экземпляр Ubuntu 24.04 LTS

```sh
PS> wsl --import Ubuntu-24.04-1 .\Ubuntu-1 .\ubuntu-24.04-wsl-root-tar.gz
```

Устанавливаем второй экземпляр Ubuntu 24.04 LTS

```sh
PS> wsl --import Ubuntu-24.04-2 .\Ubuntu-2 .\ubuntu-24.04-wsl-root-tar.gz
```

Проверяем
```sh
PS> wsl -l -v
  NAME              STATE           VERSION
* Ubuntu            Stopped         2
  Ubuntu-22.04-2    Stopped         2
  Ubuntu-22.04-1    Stopped         2
```

Подключаемся

```sh
PS> wsl -d Ubuntu-22.04-1
```

Если какой-то дистрибутив не нужен, удаляем его

```sh
PS> wsl --unregister Ubuntu-24.04-1
```

## Второй способ. Используя образ из MS Store

Смотрим список доступных дистрибутивов

```sh
PS> wsl -l -o
...
NAME                            FRIENDLY NAME
Ubuntu                          Ubuntu
Debian                          Debian GNU/Linux
kali-linux                      Kali Linux Rolling
Ubuntu-18.04                    Ubuntu 18.04 LTS
Ubuntu-20.04                    Ubuntu 20.04 LTS
Ubuntu-22.04                    Ubuntu 22.04 LTS
Ubuntu-24.04                    Ubuntu 24.04 LTS
OracleLinux_7_9                 Oracle Linux 7.9
OracleLinux_8_7                 Oracle Linux 8.7
OracleLinux_9_1                 Oracle Linux 9.1
openSUSE-Leap-15.6              openSUSE Leap 15.6
SUSE-Linux-Enterprise-15-SP5    SUSE Linux Enterprise 15 SP5
SUSE-Linux-Enterprise-15-SP6    SUSE Linux Enterprise 15 SP6
openSUSE-Tumbleweed             openSUSE Tumbleweed
```

Устанавливаем дистрибутив Ubuntu 22.04 LTS

```sh
PS> wsl --install -d Ubuntu-22.04
Запуск Ubuntu 22.04 LTS...
Installing, this may take a few minutes...
Please create a default UNIX user account. The username does not need to match your Windows username.
For more information visit: https://aka.ms/wslusers
Enter new UNIX username: user
New password:
Retype new password:
passwd: password updated successfully
Операция успешно завершена.
Installation successful!
```

После установки мы сразу авторизовались в Ubuntu. Выходим из неё

```sh
$ exit
```

Смотрим наши установленные дистрибутивы

```sh
PS> wsl -l -v
  NAME            STATE           VERSION
* Ubuntu          Stopped         2
  Ubuntu-22.04    Stopped         2
```

Создаем проект, куда будет скачен образ, и куда будут установлены дистрибутивы

```sh
PS> cd
PS> mkdir wsl
PS> cd wsl
```

Экспортируем установленный образ Ubuntu-22.04

```sh
PS> wsl --export Ubuntu-22.04 wsl-original-ubuntu
```

> Данной командой мы создаем резервную копию. Если в образе был установлен софт, развернуты проекты, они так же экспортируются.
{: .prompt-info }

Создаем новые образы, на основе того, который мы только что экспортировали

```sh
PS> wsl --import Ubuntu01 Ubuntu01 .\wsl-original-ubuntu
Выполняется импорт. Это может занять несколько минут.
Операция успешно завершена.

PS> wsl --import Ubuntu02 Ubuntu02 .\wsl-original-ubuntu
Выполняется импорт. Это может занять несколько минут.
Операция успешно завершена.
```

Проверяем

```sh
PS> wsl -l -v
  NAME            STATE           VERSION
* Ubuntu          Stopped         2
  Ubuntu02        Stopped         2
  Ubuntu-22.04    Stopped         2
  Ubuntu01        Stopped         2
```

Подключаемся под нашим пользователем `user`

```sh
PS> wsl -d Ubuntu01 --user user
```
