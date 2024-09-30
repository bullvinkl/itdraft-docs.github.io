---
layout: post
title: "Основные команды для работы с WSL из PowerShell"
date: "2024-09-29"
categories:
  - Development
tags:
  - wsl
  - powershell
  - ubuntu
image:
  path: /commons/google-manually.webp
  alt: "Основные команды для работы с WSL из PowerShell"
---

> **WSL** (Windows Subsystem for Linux) - это функция операционной системы Windows, которая позволяет запускать среду Linux на компьютере Windows без необходимости в отдельной виртуальной машине или двойной загрузке.
> Windows **PowerShell** - это усовершенствованная оболочка командной строки, созданная Microsoft для администрирования операционной системы Windows. Она позволяет системным администраторам выполнять задачи автоматизации, мониторинга и управления с помощью команд и сценариев.
{: .prompt-tip }

Смотрим список установленных дистрибутивов

```sh
PS> wsl --list --verbose

или

PS> wsl -l -v
  NAME      STATE           VERSION
* Ubuntu    Running         2
```

Завершить работу всех запущенных дистрибутивов

```sh
PS> wsl --shutdown
```

Завершить работу определенного дистрибутива

```sh
PS> wsl --terminate <Distribution Name>

или

PS> wsl -t <Distribution Name>
```

Проверка состояния WSL

```sh
PS> wsl --version
```

Запустить определенный дистрибутив в WSL и подключиться к нему

```sh
PS> wsl --distribution <Distribution Name>

или

PS> wsl -d <Distribution Name>
```

Запуск определенного дистрибутива и подключиться определенным пользователем

```sh
PS> wsl --distribution <Distribution Name> --user <User Name>
```

Определение IP-адреса

```sh
PS> wsl hostname -I
```

Подключение диска или устройства

```sh
PS> wsl --mount <DiskPath>
PS> wsl --mount -t <Filesystem>
```

Отключение диска

```sh
PS> wsl --unmount <DiskPath>
```

Смотреть список доступных дистрибутивов Linux в магазине MS Store

```sh
PS> wsl --list --online

или

PS> wsl -l -o
```

Установить дистрибутив Linux

```sh
PS> wsl --install -d <Distribution Name>
```

Назначить дефолтный дистрибутив

```sh
PS> wsl --setdefault Ubuntu
```

Не запускать дистрибутив после установки

```sh
PS> wsl --install -d <Distribution Name> --no-launch

или

PS> wsl --install -d <Distribution Name> -n
```

Удалить дистрибутив или отмена регистрации

```sh
PS> wsl --unregister <Distribution Name>
```

Экспорт дистрибутива, т.е. моментальный снимок

```sh
PS> wsl --export <Distribution Name> <FileName>
```

[Официальная документация от Microsoft](https://learn.microsoft.com/ru-ru/windows/wsl/basic-commands#mount-a-disk-or-device){:target="_blank"}
