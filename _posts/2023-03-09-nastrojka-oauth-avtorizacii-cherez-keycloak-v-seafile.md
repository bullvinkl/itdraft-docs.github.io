---
title: "Настройка oAuth авторизации через Keycloak в Seafile"
date: "2023-03-09"
categories:
  - Directory-Service
  - Storage-System
tags: 
  - "centos"
  - "keycloak"
  - "ldap"
  - "linux"
  - "oauth"
  - "rocky-linux"
  - "seafile"
image:
  path: /commons/mixer_money-min-1.png
  alt: "Настройка oAuth авторизации через Keycloak в Seafile"
---

> **OAuth** — открытый протокол авторизации, который позволяет предоставить третьей стороне ограниченный доступ к защищённым ресурсам пользователя без необходимости передавать ей логин и пароль.
{: .prompt-tip }

- Список статей из категории [Keycloak](/tags/keycloak/)

- Список статей из категории [Seafile](/tags/seafile/)

## Настройка Keycloak

Настроим клиента в Keycloak: переходим в раздел Clients и создаем нового клиента (Create client)

![](/assets/img/posts/2023/03/09/image-9.png){: w="300" }
_Create client_

![](/assets/img/posts/2023/03/09/image-10.png){: w="300" }
_Настройки_

```
General Settings
Client type: OpenID Connect
Client ID: seafile #Понадобится для дальнейшей настройки Seafile

Capability config
Client authentication: On
Authentication flow: Standard flow
, Direct access grants
```

Сохраняем, переходим в настройки клиента Seafile

![](/assets/img/posts/2023/03/09/image-11.png){: w="300" }
_Настройки клиента_

![](/assets/img/posts/2023/03/09/image-12.png){: w="300" }
_Client secret_

```
Вкладка Settings:
Root URL: http://seafile.itdraft.ru
Valid redirect URIs: http://seafile.itdraft.ru/*
    /oauth/callback/

Вкладка Credentials
Client Authenticator: Client Id and Secret
Client secret: ... #Понадобится для дальнейшей настройки Seafile
```

Настройка Keycloak завершена

## Настройка SeaFile

Отредактируем конфигурационный файл `seahub_settings.py`

```sh
$ sudo nano /opt/seafile/conf/seahub_settings.py
...
# Keycloak
ENABLE_OAUTH = True
OAUTH_ENABLE_INSECURE_TRANSPORT = True

OAUTH_CLIENT_ID = "seafile" #Client ID из вкладки Settings в Keycloak
OAUTH_CLIENT_SECRET = "pass" #Ключ из вкладки Credentials в Keycloak
OAUTH_REDIRECT_URL = 'http://seafile.itdraft.ru/oauth/callback/'

OAUTH_PROVIDER_DOMAIN   = 'seafile.gge.ext'
OAUTH_AUTHORIZATION_URL = 'https://keycloak.itraft.ru:8443/realms/myrealm/protocol/openid-connect/auth'
OAUTH_TOKEN_URL         = 'https://keycloak.itraft.ru:8443/realms/myrealm/protocol/openid-connect/token'
OAUTH_USER_INFO_URL     = 'https://keycloak.itraft.ru:8443/realms/myrealm/protocol/openid-connect/userinfo'
OAUTH_SCOPE = ["profile", "email"]
OAUTH_ATTRIBUTE_MAP = {
    "id":    (False, "not used"),
    "name":  (False, "name"),
    "email": (True, "email"),
}
```

Перезапускаем Seafile

```sh
$ sudo systemctl restart seafile seahub
```

В Web-интерфейсе должна появиться ссылка "Единая точка входа", при нажатии на которую нас перебросит в интерфейс авторизации Keycloak

![](/assets/img/posts/2023/03/09/image-13.png){: w="300" }
_Единая точка входа_
