---
layout: post
title: "[Решено] Подключение к Debian по RDP учетной записью FreeIPA"
date: "2024-09-12"
categories:
  - Directory-Service
tags:
  - freeipa
  - xrdp
image:
  path: /commons/pexels-lukas-574071-scaled.jpg
  alt: "Подключение к Debian по RDP учетной записью FreeIPA"
---

> XRDP (eXternal RDP Daemon) - это свободное программное обеспечение для удаленного доступа к Linux-серверу по протоколу RDP (Remote Desktop Protocol). XRDP позволяет пользователям из других компьютеров (включая Windows) подключаться к Linux-серверу и использовать его графическое окружение, как если бы они сидели непосредственно перед монитором сервера.
{: .prompt-tip }

Для подключения по RDP к Debian используется программное обеспечение XRPD. Однако после ввода сервера в домен FreeIPA и установки программного обеспечения XRPD, авторизация по RPD централизованной учетной записью FreeIPA не происходит. Авторизация проходит только под локальной учетной записью.

Решение проблемы:
Для решения проблемы, необходимо во FreeIPA добавить службу HBAC: `xrdp-sesman` 

> FreeIPA > Политика > Управление доступом на основе узла > Службы HBAC > Добавить
{: .prompt-info }

![](/assets/img/posts/2024/09/12/freeipa-hbac1.png){: w="300" }
_Добавить службу HBAC_

![](/assets/img/posts/2024/09/12/freeipa-hbac2.png){: w="300" }
_Службы HBAC_

А дальше добавить службу `xrdp-sesman` в правило HBAC 

> FreeIPA > Политика > Управление доступом на основе узла > Правила HBAC > Добавить
{: .prompt-info }

![](/assets/img/posts/2024/09/12/freeipa-hbac3.png){: w="300" }
_Правило HBAC_

## UPD 23.09.2024

При подключении по RDP появляется ошибка

> Unable to contact settings server
> Failed to execute child process “dbus-launch”
{: .prompt-danger }

Решение:

```sh
$ sudo apt install dbus-x11
```

![](/assets/img/posts/2024/09/12/unable-to-connect.png){: w="300" }
_Failed to execute child process dbus-launch_
