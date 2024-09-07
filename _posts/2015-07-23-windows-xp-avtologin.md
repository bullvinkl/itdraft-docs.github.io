---
title: "Windows XP автологин"
date: "2015-07-23"
categories: 
  - Manuals
tags: 
  - "regedit"
  - "windows"
image:
  path: /commons/warning01.png
  alt: "Windows XP автологин"
---

> **Windows XP** была официально локализирована на русский язык и выпущена в России в 2001 году. Она была доступна в различных версиях, включая Home Edition, Professional Edition и Media Center Edition.
{: .prompt-tip }

Раздел реестра: `HKEY_LOCAL_MACHINE/SOFTWARE/Microsoft/Windows NT/CurrentVersion/Winlogon`

Строковые параметры:

- `AutoAdminLogon` - в значение (1 - вкл. , 0-выкл)
- `DefaultUserName` - в значение реального имени пользователя.
- `DefaultPassword` - в значение реального пароля пользователя.


В зависимости от материнской платы компьютера - после первого автологина, может случиться так, что операционная система сбрасывает значение `AutoAdminLogon` в `0`, делая невозможным последующий автологин.

Что бы этого избежать надо создать `*.reg` файл со следующим содержанием:

```
REGEDIT4
[HKEY_LOCAL_MACHINE/SOFTWARE/Microsoft/Windows NT/CurrentVersion/Winlogon]
"AutoAdminLogon"="1"
```

И добавить его в автозагрузку
