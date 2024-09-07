---
title: "[Решено] Настройка User federation с MS Active Directory в Keycloak"
date: "2023-03-06"
categories: 
  - Directory-Service
tags: 
  - active-directory
  - keycloak
  - linux
image:
  path: /commons/tux30-h.jpg
  alt: "[Решено] Настройка User federation с MS Active Directory в Keycloak"
---

> **Keycloak** - программное обеспечение с открытым исходным кодом, для реализации single sign-on с управлением идентификацией и управлением доступом для современных приложений и сервисов. Это программное обеспечение написано на Java и по умолчанию поддерживает протоколы федерации удостоверений SAML и OpenID Connect (OIDC)
{: .prompt-tip }

В вэб-админке Keycloak создаем новую область (`realm`), например `msad`, и переходим в раздел `User federation`. Там добавляем LDAP-провайдера

Вкладка Settings:

![](/assets/img/posts/2023/03/06/image-5.png){: w="300" }

![](/assets/img/posts/2023/03/06/image-6.png){: w="300" }

![](/assets/img/posts/2023/03/06/image-7.png){: w="300" }

```
Вкладка Settings:
Console display name: dc01.itdraft.loc
Vendor: Active Directory
Connection URL: ldap://dc01.itdraft.loc

Bind type: simple
Bind DN: CN=test2,OU=Users,OU=ITDRAFT,DC=itdraft,DC=loc
Bind credentials: mypass
Edit mode: READ_ONLY
Users DN: OU=Users,OU=ITDRAFT,DC=itdraft,DC=loc
Username LDAP attribute: sAMAccountName
RDN LDAP attribute: cn
UUID LDAP attribute: objectGUID
User object classes: person, organizationalPerson, user
```

Проверяем подключение, сохраняем

Так же можно задать параметр `User LDAP filter`, что б Keycloak подгружал только пользователей определенной группы

```
User LDAP filter: (&(objectCategory=user)(objectClass=user)(mail=*)(telephoneNumber=*)(!(userPrincipalName=*_*))(!(userAccountControl:1.2.840.113556.1.4.803:=2))(memberOf=cn=staff01,ou=Groups,dc=itdraft,dc=ru))
```

Добавляем атрибут `First name`

Вкладка Mappers

![](/assets/img/posts/2023/03/06/image-23.png){: w="300" }

```
Add mapper:
Name: first name
Mapper type: user-attribute-ldap-mapper
User Model Attribute: firstName
LDAP Attribute: givenName
Read Only: On
Always Read Value From LDAP: On
Is Mandatory In LDAP On
```

Для проверки, переходим по ссылке "Users" в левом меню, и в строке поиск вводим `*`

![](/assets/img/posts/2023/03/06/image-8-1024x630.png){: w="300" }

Как видно, пользователи подгружаются, значит все работает
