---
title: "SSO авторизация в Netbox через Keycloak"
date: "2023-05-17"
categories: 
  - Linux
  - Netbox
  - Keycloak
tags: 
  - keycloak
  - linux
  - netbox
  - oauth
  - python
  - sso
image:
  path: /commons/yf8_6borq0vorro8.webp
  alt: "SSO авторизация в Netbox через Keycloak"
---

> **Netbox** — веб приложение с открытым исходным кодом, разработанное для управления и документирования компьютерных сетей. Изначально Netbox придуман командой сетевых инженеров DigitalOcean специально для системных администраторов.
{: .prompt-tip }

- Ранее была опубликована стать по [установке Netbox]({% post_url 2023-01-24-ustanovka-netbox-v-rocky-linux-9 %})
- Tак же, ранее была опубликована статья по [установке Keycloak]({% post_url 2022-08-29-ustanovka-keycloak-i-postgesql-v-linux-centos-rocky-debian %})

## Настройка Keycloak

В вэб-админке Keycloak создаем новую область (realm), например itdraft, в ней [настраиваем либо федерацию с LDAP]({% post_url 2023-03-06-resheno-nastrojka-user-federation-s-ms-active-directory-v-keycloak %}) MS Active Directory или FreeIPA, либо просто добавляем пользователей.

Настроим клиента в Keycloak: переходим в раздел Clients и создаем нового клиента (Create client)

![](/assets/img/posts/2023/05/17/image-50.png){: w="300" }

![](/assets/img/posts/2023/05/17/image-51.png){: w="300" }

![](/assets/img/posts/2023/05/17/image-52.png){: w="300" }

```
General Settings
Client type: OpenID Connect
Client ID: netbox # Понадобится для дальнейшей настройки SeaFile

Capability config
Client authentication: On
Authentication flow: Standard flow, Direct access grants

Login settings
Root URL: https://netbox.itdraft.ru
Home URL: https://netbox.itdraft.ru
Valid redirect URIs: https://netbox.itdraft.ru/*
```

Сохраняем, переходим в настройки клиента, копируем "секрет"

![](/assets/img/posts/2023/05/17/image-53.png){: w="300" }

```
Tab "Credentials"
Client Authenticator: Client Id and Secret
Client secret: xxx # Понадобится для дальнейшей настройки Netbox
```

Переходим во вкладку "Advanced", устанавливаем параметры в 2-х полях

![](/assets/img/posts/2023/05/17/image-54.png){: w="300" }

```
Tab "Advanced"
User info signed response algorithm: RS256
Request object signature algorithm: RS256
```

Добавляем mapper, для этого переходим во вкладку Client scopes и в активную ссылку "netbox-dedicated"

![](/assets/img/posts/2023/05/17/image-57.png){: w="300" }

![](/assets/img/posts/2023/05/17/image-55.png){: w="300" }

```
Add mapper
Mapper type: Audience
Name: netbox
Included Client Audience: netbox
Add to ID token: Yes
Add to access token: Yes
```

Осталось скопировать публичный ключ

![](/assets/img/posts/2023/05/17/image-56.png){: w="300" }

```
Realm settins > Keys > RS256 > Public key
MIIBI...AB
```

## Настройка Netbox

Переключаемся на root, активируем виртуальную среду python, устанавливаем 2 библиотеки
```sh
$ sudo su
# source /opt/netbox/venv/bin/activate
(venv) # cd /opt/netbox/netbox
(venv) # pip install django-social-auth
(venv) # pip install social-auth-core
```

Добавляем их в `local_requirements.txt`
```sh
$ sudo nano /opt/netbox/local_requirements.txt
...
django-social-auth
social-auth-core
```

Редактируем конфиг Netbox
```sh
(venv) # nano /opt/netbox/netbox/netbox/configuration.py
...
REMOTE_AUTH_ENABLED = True
REMOTE_AUTH_BACKEND = 'social_core.backends.keycloak.KeycloakOAuth2'
SOCIAL_AUTH_KEYCLOAK_KEY = 'netbox'
SOCIAL_AUTH_KEYCLOAK_SECRET = 'xx'
SOCIAL_AUTH_KEYCLOAK_PUBLIC_KEY = 'MIIBI....AB'
SOCIAL_AUTH_KEYCLOAK_AUTHORIZATION_URL = 'https://keycloak.itdraft.ru:8443/realms/itdraft/protocol/openid-connect/auth'
SOCIAL_AUTH_KEYCLOAK_ACCESS_TOKEN_URL = 'https://keycloak.itdraft.ru:8443/realms/itdraft/protocol/openid-connect/token'
SOCIAL_AUTH_KEYCLOAK_ID_KEY = 'email'
SOCIAL_AUTH_VERIFY_SSL = False
REMOTE_AUTH_HEADER = 'HTTP_REMOTE_USER'
REMOTE_AUTH_AUTO_CREATE_USER = True
REMOTE_AUTH_DEFAULT_GROUPS = []
REMOTE_AUTH_DEFAULT_PERMISSIONS = {}
...
```

Выполняем процедуру миграции
```sh
(venv) # cd /opt/netbox/netbox/
(venv) # python3 manage.py migrate
```

Деактивируем виртуальную среду Python, переключаемся на нашего пользователя, перезапускаем сервисы
```sh
(venv) # deactivate
# exit
$ sudo systemctl restart netbox netbox-rq
$ sudo systemctl status netbox
$ sudo systemctl status netbox-rq
```

![](/assets/img/posts/2023/05/17/image-58.png){: w="300" }

## Update 19.12.2023 (Ошибка после обновления Netbox)

При обновлении Netbox с версии v3.4.2 до v3.6.7 появилась ошибка:
```
TypeError: object of type 'map' has no len()
```

Решение:
```sh
$ sudo su
# source /opt/netbox/venv/bin/activate
(venv) # cd /opt/netbox/netbox
(venv) # pip uninstall python-openid
(venv) # pip uninstall python3-openid
(venv) # pip install social-auth-app-django
(venv) # nano /opt/netbox/local_requirements.txt
...
#django-social-auth
#social-auth-core
social-auth-app-django

(venv) # python3 manage.py migrate
(venv) # deactivate
$ sudo systemctl restart netbox netbox-rq
```
