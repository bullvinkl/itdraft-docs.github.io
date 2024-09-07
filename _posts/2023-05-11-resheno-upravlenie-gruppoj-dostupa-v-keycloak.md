---
title: "[Решено] Управление группой доступа в Keycloak"
date: "2023-05-11"
categories: 
  - Directory-Service
tags: 
  - "keycloak"
  - "ldap"
image:
  path: /commons/analytics_ai_x-ray.png
  alt: "Управление группой доступа в Keycloak"
---

> **Keycloak** – программное обеспечение с открытым исходным кодом, для реализации single sign-on с управлением идентификацией и управлением доступом для современных приложений и сервисов. Это программное обеспечение написано на Java и по умолчанию поддерживает протоколы федерации удостоверений SAML и OpenID Connect (OIDC)
{: .prompt-tip }

**Дано:** [Keycloak сервер]({% post_url 2022-08-29-ustanovka-keycloak-i-postgesql-v-linux-centos-rocky-debian %}) с настроенной [федерацией с MS Active Directory]({% post_url 2023-03-06-resheno-nastrojka-user-federation-s-ms-active-directory-v-keycloak %}) и [SSO Kerberos авторизацией]({% post_url 2023-04-13-resheno-sso-kerberos-avtorizacija-v-keycloak %}).
В Keycloak добавлен client. В Web-приложение добавлены настройки Keycloak клиента для авторизации по протоколу OpenID или SAML.

**Требуется:** Разрешить доступ к Web-приложению определенному списку пользователей из MS Active Directory.

Выбираем наше пространство имен (`realm`)

## В настройках клиента (во вкладке "Roles") создаем роль

![](/assets/img/posts/2023/05/11/image-49.png){: w="300" }

![](/assets/img/posts/2023/05/11/image-38.png){: w="300" }

```
Clients > myclient > Tab "Roles" > Button "Create role":
Role name: access
Description: Разрешенные пользователи
```

## Настраиваем область действия клиента (Client scopes):

- Выбираем на область, которая необходима нашему клиентскому приложению (например email)

- Переходим во вкладку `Scope`

- Назначаем роль (`Assign role`)

- Фильтруем по клиентам

- Выбираем созданную в первом шаге роль

![](/assets/img/posts/2023/05/11/image-45.png){: w="300" }

![](/assets/img/posts/2023/05/11/image-46.png){: w="300" }

```
Client scopes > email > Tab "Scope" > Assign role > Filter by client > access
```

## Создаем группу доступа "mygroup"

![](/assets/img/posts/2023/05/11/image-41.png){: w="300" }

```
Groups > Create group > mygroup
```

## Настраиваем группу доступа. Добавляем пользователей

![](/assets/img/posts/2023/05/11/image-42.png){: w="300" }

```
Groups > mygroup > Tab "Members" > Add member
```

## Настраиваем группу доступа. Добавляем сопоставление ролей (Role mapping)

- Переходим во вкладку `Role mapping`

- Назначаем роль (`Assign role`)

- Фильтруем по клиентам

- Выбираем созданную в первом шаге роль

![](/assets/img/posts/2023/05/11/image-47.png){: w="300" }

![](/assets/img/posts/2023/05/11/image-48.png){: w="300" }

```
Groups > mygroup > Tab "Role mapping" > Assign role > add "it_access"
```
