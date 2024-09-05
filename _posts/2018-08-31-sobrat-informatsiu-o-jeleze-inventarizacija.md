---
title: "Собрать информацию о железе, инвентаризация"
date: "2018-08-31"
categories: 
  - Windows
tags: 
  - "cmd"
  - "psexec"
  - "wmic"
image:
  path: /commons/pexels-lukas-574071-scaled.jpg
  alt: "инвентаризация"
---

> **PsExec** - это портативный инструмент от Microsoft, позволяющий удаленно запускать процессы с использованием учетных данных любого пользователя на удаленном компьютере Windows.

Потребовалось собрать информацию о комплектующих на ПК пользователей в домене

Для начала пишем CMD-скрипт, который будет собирать следующую информацию:

- Процессор: Модель, частота
- Материнская плата:  Производитель, модель  
    
- Оперативная память: Количество планок, объем каждой планки, общий объем, производитель  
    
- Жесткий диск: Количество, производитель, модель
- Видеокарта: Производитель, объем памяти, модель
- Операционная система: Название, Имя ПК, Сервис пак

Содержимое скрипта `hardware_info.cmd`

```
@ECHO OFF
SETLOCAL ENABLEDELAYEDEXPANSION
FOR /F "skip=1 tokens=1 delims= " %%A IN ('wmic os get CSName /Format:table') DO (
	IF %%A GTR 0 (
		SET name=%%A
		SET saveto=\\192.168.1.99\share\inv
		)
)

rem # Процессор: Текущая частота, модель, ID, Максимальная частота, модель процессора, статус
rem wmic cpu get CurrentClockSpeed, Caption, DeviceID,MaxClockSpeed, name, status /format:table > %saveto%\%name%.txt
wmic cpu get Name, CurrentClockSpeed  /format:table > %saveto%\%name%.txt

rem # BIOS: Название, серийник, версия
wmic bios get name, serialnumber, version /format:list >> %saveto%\%name%.txt

rem # Материнская плата: Производитель, Название, номер детали, модель, серийник
wmic baseboard get Manufacturer, product, Name, PartNumber, serialnumber /format:table >> %saveto%\%name%.txt

rem # Оперативная память: Количество слотов 
wmic memphysical get MemoryDevices /format:list >> %saveto%\%name%.txt

rem # Оперативная память: объем, слот, тип, производитель, номер детали, частота модуля
wmic memorychip get Capacity,DeviceLocator,FormFactor,Manufacturer,PartNumber,Speed /format:table >> %saveto%\%name%.txt

rem # Оперативная память: Польный объем установленной памяти
wmic computersystem get totalphysicalmemory /format:list >> %saveto%\%name%.txt

rem # Жесткий диск: 
wmic diskdrive get MediaLoaded, MediaType, Model, Name /format:table >> %saveto%\%name%.txt

rem # Жесткий диск: Описание, Тип, Файловая система, Свободно места, Буква, Метка
wmic logicaldisk get Description, DriveType, FileSystem, FreeSpace, Name, VolumeName /format:table >> %saveto%\%name%.txt

rem # Видеокарта: Производитель, объем памяти, модель
wmic path win32_videocontroller get AdapterCompatibility, AdapterRam, Caption /format:table >> %saveto%\%name%.txt

rem # Операционная система: Название, Страна, Имя ПК, Дата установки, Серийник, СервисПак, Версия, Директория установки 
wmic os get Caption, CountryCode, CSName, InstallDate, SerialNumber, ServicePackMajorVersion, Version, WindowsDirectory /format:list >> %saveto%\%name%.txt
```

В верхней части скрипта мы вытягиваем имя пк, чтоб  в дальнейшем сохранить текстовый файл под этим именем.  
А дальше идет вытягивание необходимой информации. Данные сохраняются в расшаренную папку

Полный список переменных, которые поддерживает команда `wmic` можно посмотреть на сайте [technet microsoft](https://blogs.technet.microsoft.com/askperf/2012/02/17/useful-wmic-queries/)

Далее можно этот скрипт распространить на ПК через групповую политику, но так как эта информация будет обновляться довольно редко, я решил выполнить запуск этого скрипта через команду `psexec`, которая входит в комплект программ [PsTools](https://technet.microsoft.com/ru-ru/sysinternals/bb897553.aspx)

Необходимо получить список ПК в домене, на которых будет выполняться скрипт. Запускаем командную строку и выполняем команду

```
C:\Users\admin> net view \\192.168.1.99\share\inv\comp.txt
```

Файл сохраняется сразу в расшаренную папку, его следует отредактировать, чтоб в нем был только перечень ПК

Теперь распространим скрипт `hardware_info.cmd ` 
Для этого на доменном контроллере, куда мы ранее установили утилиту `psexec`, запускаем командную строку и выполняем команду

```
C:\Users\admin> psexec @\\192.168.1.99\share\inv\comp.txt \\192.168.1.99\share\inv\hardware_info.cmd
```

После выполнения которой в каталоге `\\192.168.1.99\share\inv` появится список текстовых файлов с информацией о железе

UPD: Столкнулся с проблемой: команда, указанная выше, не могла запустить удаленное выполнение скрипта на ПК с Windows 10. Для решения этой проблемы использовал следующую команду:

```
C:\Users\admin> psexec -u user -p password @\\192.168.1.99\share\inv\comp.txt -h -s -d -accepteula \\192.168.1.99\share\inv\hardware_info.cmd
```

где `user` и `password` - логин и пароль администратора домена  
Так же, вместо списка `@\\192.168.1.99\share\inv\comp.txt` можно использовать `\\computer_name`, что бы выполнить скрипт на определенном ПК