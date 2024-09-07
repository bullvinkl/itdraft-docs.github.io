---
title: "Работаем с FreeIPA из командной строки"
date: "2023-02-17"
categories: 
  - Linux
  - FreeIPA
tags: 
  - "centos"
  - "cli"
  - "console"
  - "freeipa"
  - "linux"
  - "rocky-linux"
image:
  path: /commons/mining_price_down-1-min.png
  alt: "Настройка oAuth авторизации через Keycloak+FreeIPA в DokuWiki"
---

> **FreeIPA** (IPA) - это комплексное решение для централизованного управления безопасностью Linux-систем, идентификацией и аутентификацией. Оно позволяет создавать многоуровневую систему управления доступом, где руководители подразделений могут добавлять пользователей в группы и настраивать доступ к ресурсам.
{: .prompt-tip }

Бывают ситуации, когда быстрее и проще выполнить какое-нибудь действие (массовые действия) с FreeIPA через терминал. Например: добавить пользователей, сбросить пароли и т.д.

Для начала авторизуемся под пользователем с правами администратора FreeIPA

```sh
$ kinit admin
Password for admin@EXAMPLE.LOCAL:
```

## Работаем с пользователями

Посмотреть информацию о пользователе / пользователях

```sh
$ ipa user-find
$ ipa user-find a.smith
```

Заблокировать / Разблокировать пользователя

```sh
$ ipa user-disable a.smith
$ ipa user-enable a.smith
```

Добавить пользователя

```sh
$ ipa user-add a.smith --first=Adam --last=Smith --password
Password: 
Enter Password again to verify: 
```

Добавить пользователя, указав дополнитлельные атрибуты

```sh
$ ipa user-add a.smith --first="Adam Sergeevich" --last=Smith --email=a.smith@gmail.com --homedir=/home/lvas --password
```

Редактируем пользователя, добавляем атрибуты

```sh
$ ipa user-show a.smith
$ ipa user-mod a.smith --addattr=l="Moscow" --email=a.smith@mail.ru
```

Смена пароля пользователя

```sh
$ ipa user-mod a.smith --password
```

Удалить пользователя "в корзину", т.е. с возможностью восстановления

```sh
$ ipa user-del --preserve a.smith
```

Восстановить пользователя "из корзины"

```sh
$ ipa user-undel a.smith
```

Удалить несколько пользователей

```sh
$ ipa user-del --continue a.smith1 a.smith2 a.smith3
```

Доступные команды для работы с пользователями

```sh
$ ipa help user
```

## Убираем автоматический запрос смены пароля

Во FreeIPA при добавлении пользователя и последующей его авторизации (в админке FreeIPA или по SSH) запрашивается принудительная смена пароля. Иногда возникают ситуации, когда надо отключить принудительную смену пароля.

```sh
$ ipa user-mod a.smith --setattr=krbPasswordExpiration=20230817011529Z
```

## Работаем с группами пользователей

Создать новую группу пользователей

```sh
$ ipa group-add g_test --desc="Test users"
$ ipa group-add global_admins --desc="Global admins"
```

Добавить пользователей в группу

```sh
$ ipa group-add-member g_test --users=a.smith
$ ipa group-add-member global_admins --users=a.smith
```

Массовое добавление пользователей в группу

```sh
$ ipa group-add-member ws_helpdesk --users={lvas,ppv,ipetrov}
```

Удалить пользователя из группы

```sh
$ ipa group-remove-member ws_helpdesk --users=ipetrov
```

Посмотреть информацию о группе

```sh
$ ipa group-find
$ ipa group-find g_test
```

Доступные команды для работы с группами

```sh
$ ipa help group
```

## Работаем с политиками паролей

Посмотреть доступные политики паролей

```sh
$ ipa pwpolicy-show
```

Добавить политику паролей глобальных адинов

```sh
$ ipa pwpolicy-add global_admins --maxlife=90 --minlife=1 --history=10 --minclasses=0 --minlength=14 --maxfail=6 --failinterval=60 --lockouttime=600 --priority=1
```

Добавить политику паролей пользователей

```sh
$ ipa pwpolicy-add ipausers --maxlife=180 --minlife=1 --history=3 --minclasses=0 --minlength=8 --maxfail=6 --failinterval=60 --lockouttime=600 --priority=10
```

Редактировать политику паролей

```sh
$ ipa pwpolicy-mod global_admins --maxfail=6
```

Посмотреть политику паролей / политику паролей для пользователя

```sh
$ ipa pwpolicy-show global_admins
$ ipa pwpolicy-show --user=a.smith
```

Доступные команды для работы с политиками паролей

```sh
$ ipa help pwpolicy
```

## Запрет авторизации по истечению срока действия пароля

В политиках паролей пользователей FreeIPA есть такой параметр как `Grace login limit` - льготный лимит входа.

Льготный период определяет количество входов в систему (web, ssh, LDAP), разрешенных после истечения срока действия.  
Значение `-1` означает, что авторизация пользователя не блокируется при истечение срока действия. Т.е. при истечении срока действия пароля пользователь может сменить пароль через админку FreeIPA или при подключении по SSH.

Чтобы запретить авторизацию пользователей (в том числе и LDAP) по истечению срока действия пароля, необходимо установить этот параметр `= 0`. 
Но что б не заблокировать себе админский доступ к FreeIPA необходимо создать политику паролей для админов, со значением `Grace Login Limit = -1`

## Работаем с политиками HBAC

Посмотреть политики HBAC

```sh
$ ipa hbacrule-find
```

Отключаем HBAC политику `allow_all`

```sh
$ ipa hbacrule-disable allow_all
```

Добавить HBAC сервис

```sh
$ ipa hbacsvc-add wikiapp
```

Добавить HBAC политику

```sh
$ ipa hbacrule-add allow_wikiapp
```

Добавить в HBAC политику HBAC сервис

```sh
$ ipa hbacrule-add-service allow_wikiapp --hbacsvcs=wikiapp
```

Доступные команды для работы с политиками / сервисами HBAC

```sh
$ ipa help hbacrule
$ ipa help hbacsvc
```

## Глобальные настройки FreeIPA

Проверьте настройки по умолчанию

```sh
$ ipa config-show
```

Используем Bash как командный интерпретатор по умолчанию

```sh
$ ipa config-mod --defaultshell=/bin/bash
```
