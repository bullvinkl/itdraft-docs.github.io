---
title: "Резервное копирование (бэкап) базы 1С, лог на почту"
date: "2016-03-23"
categories: 
  - Automation
tags: 
  - cmd
  - backup
  - forfiles
  - mailsend
image:
  path: /commons/merger-min.png
  alt: "Резервное копирование базы 1С"
---

> **Резервное копирование (бэкап)**: это процесс создания точной копии данных, конфигурации или приложения, хранящейся отдельно от оригинала. Это обеспечивает защиту данных в случае непредвиденных ситуаций, таких как чрезвычайные ситуации, человеческий фактор, угрозы безопасности или системные сбой.
{: .prompt-tip }

Для создания резервных копий нам понадобится:

1. Архиватор 7-Zip
2. Утилита `forfiles` - консольная утилита Windows для операций с файлами, которая уже присутствует в стандартной поставке в Windows 7 и WS2008R2. Позволяет производить поиск по маске и/или возрасту и применять действия к найденным файлам.
3. Утилита [CmdEmail](https://cmdemail.codeplex.com/) - утилита для отправки email-сообщений через командную строку.

Создаем скрипт C:\backup\scripts\backup.cmd

```
@echo off
:: дата в имени файлов
set t=%date:~6,4%-%date:~3,2%-%date:~0,2%
:: тут лежит утилита forfiles и CmdEmail
set f=C:\backup\scripts
:: тут лежат базы
set from=Z:\
:: сюда будем сохранять архив
set to=Y:\1C
:: шаг_1 архивируем и пишем лог
"%programfiles(x86)%\7-Zip\7z.exe" a -tzip -ssw -mx7 "%to%\1C_BUH_%t%.zip" "%from%\1C_BUH" | findstr /P /I /V "Compressing Scanning 7-Zip" >> %to%\log_%t%.txt
:: шаг_2 ищем файлы старше 7 дней и пишем их лог, ищем файлы старше 7 дней и удаляем их
"%f%\forfiles.exe" -p %to% -s -m *.* -D -7 >> %to%\log_%t%.txt | "%f%\forfiles.exe" -p %to% -s -m *.* -D -7 -c "cmd /c del /q @path"
:: шаг_3 отправляю лог на почту
"%f%\CmdEmail.exe" -f "from@mailserver.ru" -t "toadmin@mailserver.ru" -s "1c backup log %t%" -b "Log:" -a "%to%\log_%t%.txt"
```

Скачиваем утилиты, указанные выше. В файле конфигурации `CmdEmail.exe.config` задаем параметры подключения к email'у

Далее данный скрипт прописываем в планировщике заданий

## UPD 30.11.2018

Для того, что бы отправлять лог на почтовый ящик через ssl smtp (порт 465), вместо утилиты CmdEmail можно использовать другую утилиту: [mailsend](https://github.com/muquit/mailsend)

Тогда в скрипте выше шаг 3 будет следующим:

```bash
:: шаг_3 отправляю лог на почту
set "mailsender=%f%\mailsend1.19.exe"
set "smtpserver=mail.example.com"
set "smtpport=465"
set "smtpuser=from@mailserver.ru"
set "smtppwd=123456"
set "smtpsender=toadmin@mailserver.ru" 
set "subject=1c backup log %t%"
set "body=Log:"
"%mailsender%" -smtp "%smtpserver%" -port "%smtpport%" -ssl -auth -user "%smtpuser%" -pass "%smtppwd%" -f "%smtpuser%" -t "%smtpsender%" -name "%smtpuser%" -rt "%smtpuser%" +cc +bc -q -sub "%subject%" -M "%body%"  -attach "%to%\log_%t%.txt"
```
