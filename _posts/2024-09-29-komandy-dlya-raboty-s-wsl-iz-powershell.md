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

```powershell
PS> wsl --list --verbose

или

PS> wsl -l -v
  NAME      STATE           VERSION
* Ubuntu    Running         2
```

Завершить работу всех запущенных дистрибутивов

```powershell
PS> wsl --shutdown
```

Завершить работу определенного дистрибутива

```powershell
PS> wsl --terminate <Distribution Name>

или

PS> wsl -t <Distribution Name>
```

Проверка состояния WSL

```powershell
PS> wsl --version
```

Запустить определенный дистрибутив в WSL и подключиться к нему

```powershell
PS> wsl --distribution <Distribution Name>

или

PS> wsl -d <Distribution Name>
```

Запуск определенного дистрибутива и подключиться определенным пользователем

```powershell
PS> wsl --distribution <Distribution Name> --user <User Name>
```

Определение IP-адреса

```powershell
PS> wsl hostname -I
```

Подключение диска или устройства

```powershell
PS> wsl --mount <DiskPath>
PS> wsl --mount -t <Filesystem>
```

Отключение диска

```powershell
PS> wsl --unmount <DiskPath>
```

Смотреть список доступных дистрибутивов Linux в магазине MS Store

```powershell
PS> wsl --list --online

или

PS> wsl -l -o
```

Установить дистрибутив Linux

```powershell
PS> wsl --install -d <Distribution Name>
```

Назначить дефолтный дистрибутив

```powershell
PS> wsl --setdefault Ubuntu
```

Не запускать дистрибутив после установки

```powershell
PS> wsl --install -d <Distribution Name> --no-launch

или

PS> wsl --install -d <Distribution Name> -n
```

Удалить дистрибутив или отмена регистрации

```powershell
PS> wsl --unregister <Distribution Name>
```

Экспорт дистрибутива, т.е. моментальный снимок

```powershell
PS> wsl --export <Distribution Name> <FileName>
```

[Официальная документация от Microsoft](https://learn.microsoft.com/ru-ru/windows/wsl/basic-commands#mount-a-disk-or-device){:target="_blank"}
