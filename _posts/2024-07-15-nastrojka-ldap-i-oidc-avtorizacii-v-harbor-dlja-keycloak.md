---
layout: post
title: "Настройка LDAP и OIDC авторизации в Harbor для Keycloak"
date: "2024-07-15"
categories:
  - Linux
  - Harbor
  - Keycloak
tags:
  - debian
  - freeipa
  - harbor
  - keycloak
  - oidc
  - rocky-linux
image:
  path: /commons/pexels-christina-morillo-1181244-scaled.jpg
  alt: "Настройка LDAP и OIDC авторизации в Harbor для Keycloak"
---

> **Харбор** — искусственный остров в Сиэтле, на котором расположена судостроительная верфь.  
> Ой, т.е. **Harbor** — это бесплатный реестр для хранения Docker образов c [открытым исходным кодом](https://github.com/goharbor/harbor), который предоставляет доступ к образам с помощью политик, а также умеет сканировать образы на наличие уязвимостей.

## Установка Harbor

Установка Harbor достаточно легкая. Для начал [устанавливаем Docker]({% post_url 2022-11-15-svyazka-wordpress-v-docker-subd-lokalno-v-debian-11 %})

Скачиваем дистрибутив Harbor и распаковываем его
```sh
$ wget https://github.com/goharbor/harbor/releases/download/v2.11.0/harbor-online-installer-v2.11.0.tgz
$ tar -zxvf harbor-online-installer-v2.11.0.tgz
```

Переходим в директорию, копируем и правим конфиг
```sh
$ cd harbor
$ cp harbor.yml.tmpl harbor.yml
$ nano harbor.yml
```

Дефолтный hostname надо заменить на любой другой

Запускаем установку
```sh
$ sudo ./install.sh
...
✔ ----Harbor has been installed and started successfully.----
```

На этом установка Harbor завершена

## Настройка LDAP авторизации

В качестве LDAP-сервера будем использовать IDM (FreeIPA). Для настройки LDAP-авторизации авторизуемся в web-интерфейсе Harbor (Administration > Configuration)

![](/assets/img/posts/2024/07/15/image-3.png)
_-_Administration > Configuration_

Параметры:
```
LDAP URL: ldap://freeipa.itdraft.ru
LDAP Search DN: uid=10.harbor_s,cn=users,cn=accounts,dc=itdraft,dc=ru
LDAP Search Password: %superpassword%
LDAP Base DN: cn=users,cn=accounts,dc=itdraft,dc=ru
LDAP Filter: memberOf=cn=g_registry,cn=groups,cn=accounts,dc=itdraft,dc=ru
LDAP UID: uid
```

Настаиваем сетевую связанность с LDAP-сервером, проверяем авторизацию

## OIDC авторизация. Настройка Keycloak

На стороне Keycloak выбираем нужный Realm, создаем клиент со следующими параметрами

![](/assets/img/posts/2024/07/15/image-1.png)
_Settings 1_

![](/assets/img/posts/2024/07/15/image-2.png)
_Settings 2_

Во вкладке Settings нам понадобится `ClientID`
Во вкладке Credentials понадобится `Client secret`

Переходим во вкладку Client scopes > harbor2-dedicated и создаем новый Mapper

![](/assets/img/posts/2024/07/15/image-4.png)
_Client scopes_

![](/assets/img/posts/2024/07/15/image-5.png)
_Mapper_

![](/assets/img/posts/2024/07/15/image-6.png)
_Group Membership_

Параметры:
```
Mapper type: Group Membership
Name: groups
Token Claim Name: groups
Full group path: Off
Add to ID token: On
Add to access token: On
Add to userinfo: On
```

Этот mapper нам понадобится для назначения администраторов Harbor

Переходим в пункт меню Groups, создаем группу админов (harbor-admin и группу для доступа к проекту (у меня - express), добавляем в эти группы пользователей

![](/assets/img/posts/2024/07/15/image-7.png)
_создаем группу админов_

На этом настройки на стороне Keycloak закончены

## OIDC авторизация. Настройка Harbor

Авторизуемся в web-интерфейсе Harbor (Administration > Configuration), выбираем Auth Mode: OIDC

Если выпадающее меню для переключения типа авторизации неактивно, удаляем всех добавленных пользователей

![](/assets/img/posts/2024/07/15/image-8.png)
_Auth Mode: OIDC_

Параметры:
```
Auth Mode: OIDC   # Тип авторизации
OIDC Provider Name: KEYCLOAK   # Наименование кнопки в админке
OIDC Endpoint: https://keycloak.itdraft.ru:8443/realms/itdraft   # URL до нашего Realm
OIDC Client ID: harbor2  # Создавали в Keycloak
OIDC Client Secret: %token%  # Копировали в Keycloak из вкладки Credentials
OIDC Group Filter: (express|harbor)   # Группы, для назначения польщзователей в проект, поддерживаются регулярки
Group Claim Name: groups   # Создавали в Keycloak (поле Token Claim Name)
OIDC Admin Group: harbor-admin  # Создавали группу в Keycloak
OIDC Scope: openid,profile,email,offline_access,roles   # roles - для назначения групп, offline_access - помогает с авторизацией из Docker
Automatic onboarding: On   # Автоматическое назначение username для новых пользователей
Username Claim: preferred_username   # какое поле будет подставляться. Можно так же name, first_name, last_name
```

Как ограничить доступ к Harbor группой доступа из Keycloak я пока что не нашел

Для получения токена авторизации в Harbor через Docker, авторизуемся в web-интерфейсе Harbor, usename > User Profile, копируем CLI secret

![](/assets/img/posts/2024/07/15/image-9.png){: h="300" }
_User Profile_

![](/assets/img/posts/2024/07/15/image-10.png){: h="300" }
_CLI secret_