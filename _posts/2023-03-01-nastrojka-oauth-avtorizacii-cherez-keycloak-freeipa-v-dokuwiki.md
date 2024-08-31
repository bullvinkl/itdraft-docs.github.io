---
title: "Настройка oAuth авторизации через Keycloak+FreeIPA в DokuWiki"
date: "2023-03-01"
categories: 
  - Linux
  - Keycloak
  - FreeIPA
  - DokuWiki
tags: 
  - centos
  - debian
  - dokuwiki
  - freeipa
  - keycloak
  - linux
  - oauth
  - rocky-linux
image:
  path: /commons/transformers-min.png
  alt: "Настройка oAuth авторизации через Keycloak+FreeIPA в DokuWiki"
---

> **OAuth** — открытый протокол авторизации, который позволяет предоставить третьей стороне ограниченный доступ к защищённым ресурсам пользователя без необходимости передавать ей логин и пароль.

- [Установка Keycloak]({% post_url 2022-08-29-ustanovka-keycloak-i-postgesql-v-linux-centos-rocky-debian %}) была рассмотрена в одной из предыдущих статей.

- [Установка DokuWiki]({% post_url 2022-08-09-ustanovka-dokuwiki-nginx-php-fpm-v-debian-11-bullseye %}) так же была рассмотрена в одной из предыдущих статей.

## Настройка User federation с FreeIPA в Keycloak

В вэб-админке Keycloak создаем новую область (realm), например `myrealm`, и переходим в раздел `User federation`. Там добавляем LDAP-провайдера

Вкладка Settings:

![](/assets/img/posts/2023/03/01/key-ldap1.png){: w="300" }

![](/assets/img/posts/2023/03/01/key-ldap2.png){: w="300" }

```
Вкладка Settings:
Console display name: freeipa.itdraft.ru
Vendor: Red Hat Directory Server
Connection URL: ldap://freeipa.itdraft.ru

# Во FreeIPA должен быть заведен пользователей keycloak с паролем mypass
Bind DN: uid=keycloak,cn=users,cn=accounts,dc=itdraft,dc=ru
Bind credentials: mypass
Edit mode: READ_ONLY
Users DN: cn=users,cn=accounts,dc=itdraft,dc=ru
Username LDAP attribute: uid
RDN LDAP attribute: uid
UUID LDAP attribute: uid
User object classes: inetOrgPerson, organizationalPerson

# Разрешаем доступ для группы gr-keycloak
User LDAP filter: (&(objectclass=person)(uid=*)(memberOf=cn=gr-keycloak,cn=groups,cn=accounts,dc=itdraft,dc=ru))
```

Проверяем подключение, сохраняем

Вкладка Mappers > first name

![](/assets/img/posts/2023/03/01/image.png){: w="300" }

```
LDAP Attribute: givenname
```

Сохраняем

Создаем клиента для dokuwiki: переходим в раздел Clients и создаем нового клиента (Create client)

![](/assets/img/posts/2023/03/01/doku-key-create-1-1.png){: w="300" }

![](/assets/img/posts/2023/03/01/doku-key-create-2-1.png){: w="300" }

```
General Settings
Client type: OpenID Connect
Client ID: dokuwiki #Понадобится для настройки плагина в dokuwiki

Capability config
Client authentication: On
Authentication flow: All
```

Сохраняем, переходим в настройки клиента dokuwiki

![](/assets/img/posts/2023/03/01/image-1.png){: w="300" }

![](/assets/img/posts/2023/03/01/image-2.png){: w="300" }

```
Вкладка Settings
Valid redirect URIs: https://dokuwiki.itdraft.ru/*

Вкладка Credentials
Client Authenticator: Client Id and Secret
Client secret: ... #Понадобится для настройки плагина в dokuwiki
```

Настройка Keycloak завершена

## Настройка DokuWiki

Авторизуемся в Dokuwiki, переходим в Управление > Управление дополнениями > Поиск и установка

Ищем дополнение: oAuth

![](/assets/img/posts/2023/03/01/image-3.png){: w="300" }

Устанавливаем плагины:

- oauth plugin

- oAuth Keycloak Service

Переходим в Управление > Настройки вики

```
Механизм аутентификации: oauth
...
Oauth
Register authenticated users even if self-registration is disabled in main configuration: yes
...
Oauthkeycloak
Client ID: dokuwiki #Из настроек клиента в Keycloak
Cient Secret: ... #Из настроек клиента в Keycloak
OpenID Connect Auto Discovery URL: https://keycloak.itdraft.ru:8443/realms/myrealm/.well-known/openid-configuration
```

![](/assets/img/posts/2023/03/01/image-4.png){: w="300" }

Переходим в профиль пользователя и разрешаем авторизацию через Keycloak