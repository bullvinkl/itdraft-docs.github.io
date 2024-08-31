---
title: "[Решено] SSO Kerberos авторизация в Keycloak"
date: "2023-04-13"
categories: 
  - Keycloak
  - Linux
  - Active-Directory
tags: 
  - centos
  - debian
  - kerberos
  - keycloak
  - linux
  - rocky-linux
  - sso
  - keytab
image:
  path: /commons/install-nix-os-in-vm.webp
  alt: "SSO Kerberos авторизация в Keycloak"
---

> **Kerberos** — сетевой протокол аутентификации, который предлагает механизм взаимной аутентификации клиента и сервера перед установлением связи между ними. Kerberos выполняет аутентификацию в качестве службы аутентификации доверенной третьей стороны, используя криптографический разделяемый секрет, при условии, что пакеты, проходящие по незащищенной сети, могут быть перехвачены, модифицированы и использованы злоумышленником. Kerberos построен на криптографии симметричных ключей и требует наличия центра распределения ключей.

- Ранее была опубликована статья по [настройке федерации с Microsift Active Directory]({% post_url 2023-03-06-resheno-nastrojka-user-federation-s-ms-active-directory-v-keycloak %})

В Active Directory для нашего пользователя keycloak создаем `spn-запись` и `keytab-файл`:

```
setspn -A HTTP/keycloak.itdraft.ru@ITDRAFT.RU keycloak

ktpass -princ HTTP/keycloak.itdraft.ru@ITDRAFT.RU -mapUser keycloak@ITDRAFT.RU -crypto RC4-HMAC-NT -pass * -kvno 0 -ptype KRB5_NT_PRINCIPAL -out C:\tmp\keycloak.keytab
```

Копируем `keytab-файл` на сервер с установленным keycloak в раздел `/opt/keycloak`

Переходим в раздел User federation и редактируем LDAP-провайдера

![](/assets/img/posts/2023/04/13/image-35.png){: w="300" }

```
Kerberos integration:
Allow Kerberos authentication: On
Kerberos realm: ITDRAFT.RU
Server principal: HTTP/keycloak.itdraft.ru@ITDRAFT.RU
Key tab: /opt/keycloak/keycloak.keytab
Debug: On # Можно включить на время отладки
```

Для дебага, мониторим логи keycloak `/tmp/keycloak.log`

Через групповую политику добавляем наш домен keycloak.itdraft.ru в доверенные

Для Chrome, Edge:

```
AuthServerWhitelist = keycloak.itdraft.ru
```

Для Firefox:

```
network.negotiate-auth.trusted-uris
network.automatic-ntlm-auth.trusted-uris
```