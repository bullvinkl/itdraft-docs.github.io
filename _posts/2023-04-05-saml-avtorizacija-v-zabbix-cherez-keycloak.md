---
title: "SAML-авторизация в Zabbix через Keycloak"
date: "2023-04-05"
categories: 
  - Linux
  - Zabbix
  - Keycloak
tags: 
  - "keycloak"
  - "linux"
  - "saml"
  - "zabbix"
image:
  path: /commons/data_leak.jpg
  alt: "SAML-авторизация в Zabbix через Keycloak"
---

> **Keycloak** – продукт с открытым кодом для реализации single sign-on с возможностью управления доступом, нацелен на современные применения и сервисы. По состоянию на 2018 год, этот проект сообщества JBoss находится под управлением Red Hat которые используют его как upstream проект для своего продукта RH-SSO

Интеграция Zabbix и Keycloak по протоколу SAML 2.0 производилась со следующими версиями ПО:

- Zabbix 6.2
- Keycloak 19.0.1

**Исходные данные:** у нас уже создана новая область (Realm), и в ней настроена федерация со службой каталогов ([MS Active Directory]({% post_url 2023-03-06-resheno-nastrojka-user-federation-s-ms-active-directory-v-keycloak %}) или [FreeIPA]({% post_url 2023-03-01-nastrojka-oauth-avtorizacii-cherez-keycloak-freeipa-v-dokuwiki %}))

## Настройка Keycloak

Создаем клиента для Zabbix: переходим в раздел Clients и создаем нового клиента (Create client)

![](/assets/img/posts/2023/04/05/image-1.png){: w="300" }
_Создаем клиента_

```
Create client:
Client type: SAML
Client ID: zabbix
Name: Zabbix
```

Сохраняемся, редактируем остальные настройки

**Tab Settings > Access settings**

![](/assets/img/posts/2023/04/05/image-2.png){: w="300" }

```
Access settings:
Root URL: https://zabbix.itdraft.ru/
Home URL: https://zabbix.itdraft.ru/
Valid redirect URIs: https://zabbix.itdraft.ru/*
IDP-Initiated SSO URL name: zabbix.itdraft.ru   # Ниже появится URL, который нам понадобится для настройки Zabbix
Master SAML Processing URL: https://zabbix.itdraft.ru/index_sso.php?acs
```

**Tab Settings > SAML capabilities**

![](/assets/img/posts/2023/04/05/image-3.png){: w="300" }

```
SAML capabilities:
Name ID format: username
Force POST binding: On
Include AuthnStatement: On
```

**Tab Settings >** **Signature and Encryption, Login settings, Logout settings**

![](/assets/img/posts/2023/04/05/image-4.png){: w="300" }

```
Signature and Encryption:
Sign documents: On
Signature algorithm: RSA_SHA256
SAML signature key name: KEY_ID
Canonicalization method: EXCLUSIVE

Login settings:
Login theme: keycloak

Logout settings:
Front channel logout: On
```

**Tab Key > Signing keys config**

![](/assets/img/posts/2023/04/05/image-5.png){: w="300" }

```
Signing keys config:
Client signature required: Off

Encryption keys config:
Encrypt assertions Off
```

**Tab Client Scope > zabbix-dedicate > Tab Mappers**

Add mapper > By configuration > User Property: Name

![](/assets/img/posts/2023/04/05/image-6.png){: w="300" }

```
Mapper type: User Property
Name: name
Property: Username
Friendly Name: Username
SAML Attribute Name: username
SAML Attribute NameFormat: Basic
```

Add mapper > By configuration > User Property: Email

![](/assets/img/posts/2023/04/05/image-7.png){: w="300" }

```
Mapper type: User Property
Name: email
Property: Email
Friendly Name: Email
SAML Attribute email
SAML Attribute NameFormat: Basic
```

Add mapper > By configuration > User Property: First Name

![](/assets/img/posts/2023/04/05/image-8.png){: w="300" }

```
Mapper type: User Property
Name: first_name
Property: FirstName
Friendly Name: FirstName
SAML Attribute first_name
SAML Attribute NameFormat: Basic
```

Add mapper > By configuration > User Property: Last Name

![](/assets/img/posts/2023/04/05/image-9.png){: w="300" }

```
Mapper type: User Property
Name: last_name
Property: LastName
Friendly Name: LastName
SAML Attribute last_name
SAML Attribute NameFormat: Basic
```

Add mapper > By configuration > Role list: Role list

![](/assets/img/posts/2023/04/05/image-21.png){: w="300" }

```
Mapper type: Role list
Name: role list
Role attribute name: Role
Friendly Name: 
SAML Attribute NameFormat: Basic
Single Role Attribute: On
```

Итого. Добавленные атрибуты:

![](/assets/img/posts/2023/04/05/image-11.png){: w="300" }

**Tab Advanced > Fine Grain SAML Endpoint Configuration**

![](/assets/img/posts/2023/04/05/image-20.png){: w="300" }

```
Fine Grain SAML Endpoint Configuration:
Assertion Consumer Service Redirect Binding URL: https://zabbix.itdraft.ru/index_sso.php?acs
```

Продолжаем настройки Keycloak

Client scopes > role_list

![](/assets/img/posts/2023/04/05/image-13.png){: w="300" }

role_list > Mappers > role list

![](/assets/img/posts/2023/04/05/image-14.png){: w="300" }

![](/assets/img/posts/2023/04/05/image-15.png){: w="300" }

```
Role list
Single Role Attribute: On
```

Копируем открытый ключ:

Realm settings > Tab keys > RS256 > Certificate

![](/assets/img/posts/2023/04/05/image-16.png){: w="300" }

так же этот ключ можно получить по ссылке:

```
https://<keycloak>/auth/realms/<myrealm>/protocol/saml/descriptor
<ds:X509Certificate> ... </ds:X509Certificate>
```

![](/assets/img/posts/2023/04/05/image-19.png){: w="300" }

## Настройка Zabbix в консоли

Создаем сертификат `idp.crt`
```sh
$ cd /usr/share/zabbix/conf/certs
$ sudo nano idp.crt
-----BEGIN CERTIFICATE-----
сюда втавляем скопированный сертификат
-----END CERTIFICATE-----

```

Создаем самоподписанные сертификат `sp.crt` и ключ `sp.key`
```sh
$ sudo openssl req -x509 -sha256 -newkey rsa:2048 -keyout sp.key -out sp.crt -days 3650 -nodes -subj '/CN=My Zabbix Server'
```

Меняем права и владельца на сертификаты и ключ
```sh
# Для Debian
$ sudo chown www-data. *
$ sudo chmod 600 *

# Для Centos
$ sudo chown nginx. *
$ sudo chmod 755 *
```

Редактируем конфиг `zabbix.conf.php`, правим в самом конце файла
```sh
$ sudo nano /etc/zabbix/web/zabbix.conf.php
...
$SSO['SP_KEY']                  = 'conf/certs/sp.key';
$SSO['SP_CERT']                 = 'conf/certs/sp.crt';
$SSO['IDP_CERT']                = 'conf/certs/idp.crt';
$SSO['SETTINGS']                = [ 'security' => [ 'requestedAuthnContext' => false ] ];
```

## Настройка Zabbix в web-итерфейсе

Переходим в настройки авторизации: Administration> Authentication > SAML settings

![](/assets/img/posts/2023/04/05/image-18.png){: w="300" }

```
Enable SAML authentication: Check
IdP entity ID: https://keycloak.itdraft.ru:8443/realms/myrealm
SSO service URL: https://keycloak.itdraft.ru:8443/realms/myrealm/protocol/saml/clients/zabbix.itdraft.ru # этот параметр сгенерировал keycloak (см. начало статьи)
Username attribute: username # этот атрибут мы задавали в Keycloak (Mappers)
SP entity ID: zabbix # название нашего клиента в Keycloak
SP name ID format: urn:oasis:names:tc:SAML:2.0:nameid-format:persistent
+ AuthN requests
+ Logout requests
+ Logout responses
Case-sensitive login: check
```

Что бы авторизация заработала, в Zabbix надо добавить пользователей.