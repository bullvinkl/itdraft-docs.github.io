---
title: "LDAP-авторизация в Netbox (AD, FreeIPA)"
date: "2023-02-01"
categories: 
  - Directory-Service
  - Asset-Management
tags: 
  - "active-directory"
  - "assetmanager"
  - "cmdb"
  - "freeipa"
  - "itsm"
  - "ldap"
  - "netbox"
image:
  path: /commons/stress-min-1.png
  alt: "LDAP-авторизация в Netbox"
---

> **Netbox** — веб приложение с открытым исходным кодом, разработанное для управления и документирования компьютерных сетей. Изначально Netbox придуман командой сетевых инженеров DigitalOcean специально для системных администраторов.
{: .prompt-tip }

## Настройка LDAP-авторизации

Устанавливаем недостающие пакеты

```sh
$ sudo yum install python-devel
```

Переключаемся на пользователя root и активируем виртуальную среду Python

```sh
$ sudo su
# source /opt/netbox/venv/bin/activate
```

Устанавливаем python-модуль и деактивируем виртуальную среду

```sh
(venv) # pip install django-auth-ldap
(venv) # deactivate
```

Переключаемся на нашего основного пользователя

```sh
# exit
```

Добавляем установленный python-модуль в файл `local_requirements.txt`

```sh
$ sudo nano /opt/netbox/local_requirements.txt
...
django-auth-ldap
```

Меняем конфигурационный файл `configuration.py`

```sh
$ sudo nano /opt/netbox/netbox/netbox/configuration.py
...
#REMOTE_AUTH_ENABLED =  False
REMOTE_AUTH_ENABLED =  True
#REMOTE_AUTH_BACKEND = 'netbox.authentication.RemoteUserBackend'
REMOTE_AUTH_BACKEND = 'netbox.authentication.LDAPBackend'
...
```

Создаем конфигурационный файл с настройками LDAP и меняем владельца файла

```sh
$ sudo touch /opt/netbox/netbox/netbox/ldap_config.py
$ sudo chown netbox:netbox /opt/netbox/netbox/netbox/ldap_config.py
```

## LDAP авторизации для FreeIPA

Пример файла LDAP конфигурации `ldap_config.py` для FreeIPA

```sh
$ sudo nano /opt/netbox/netbox/netbox/ldap_config.py

import ldap
from django_auth_ldap.config import LDAPSearch, NestedGroupOfNamesType


AUTH_LDAP_SERVER_URI = "ldap://172.16.0.2"

AUTH_LDAP_CONNECTION_OPTIONS = {
    ldap.OPT_REFERRALS: 0
}

AUTH_LDAP_BIND_DN = "uid=netbox_s,cn=users,cn=accounts,dc=itdraft,dc=local"
AUTH_LDAP_BIND_PASSWORD = "qus*MssscCCn3r"

LDAP_IGNORE_CERT_ERRORS = True

AUTH_LDAP_USER_SEARCH = LDAPSearch("cn=users,cn=accounts,dc=itdraft,dc=local", ldap.SCOPE_SUBTREE, "(uid=%(user)s)")

AUTH_LDAP_USER_DN_TEMPLATE = None

AUTH_LDAP_USER_ATTR_MAP = {
    "first_name": "givenName",
    "last_name": "sn",
    "email": "mail"
}

AUTH_LDAP_GROUP_SEARCH = LDAPSearch("cn=groups,cn=accounts,dc=itdraft,dc=local", ldap.SCOPE_SUBTREE, "(objectClass=groupOfNames)")

#AUTH_LDAP_GROUP_TYPE = NestedGroupOfNamesType()
AUTH_LDAP_GROUP_TYPE = NestedGroupOfNamesType(name_attr="cn")

AUTH_LDAP_ALWAYS_UPDATE_USER = True

#AUTH_LDAP_MIRROR_GROUPS = True

AUTH_LDAP_USER_FLAGS_BY_GROUP = {
    "is_active": "cn=ipausers,cn=groups,cn=accounts,dc=itdraft,dc=local",
    "is_staff": "cn=ipausers,cn=groups,cn=accounts,dc=itdraft,dc=local",
    "is_superuser": "cn=ipausers,cn=groups,cn=accounts,dc=itdraft,dc=local"
}

AUTH_LDAP_FIND_GROUP_PERMS = True

AUTH_LDAP_CACHE_TIMEOUT = 3600

AUTHENTICATION_BACKENDS = (
    "django_auth_ldap.backend.LDAPBackend",
    "django.contrib.auth.backends.ModelBackend",
)
```

## LDAP авторизации для Active Directory

Пример файла LDAP конфигурации `ldap_config.py` для Active Directory

```sh
$ sudo nano /opt/netbox/netbox/netbox/ldap_config.py

import ldap
from django_auth_ldap.config import LDAPSearch, NestedGroupOfNamesType


AUTH_LDAP_SERVER_URI = "ldap://172.16.0.2"

AUTH_LDAP_CONNECTION_OPTIONS = {
    ldap.OPT_REFERRALS: 0
}

AUTH_LDAP_BIND_DN = "cn=netbox_s,ou=ServiceAccounts,ou=MSK,dc=itdraft,dc=ru"
AUTH_LDAP_BIND_PASSWORD = "3ghJnd56hff"

LDAP_IGNORE_CERT_ERRORS = True

AUTH_LDAP_USER_SEARCH = LDAPSearch("ou=Users,ou=MSK,dc=itdraft,dc=ru", ldap.SCOPE_SUBTREE, "(sAMAccountName=%(user)s)")

AUTH_LDAP_USER_DN_TEMPLATE = None

AUTH_LDAP_USER_ATTR_MAP = {
    "first_name": "givenName",
    "last_name": "sn",
    "email": "mail"
}

AUTH_LDAP_GROUP_SEARCH = LDAPSearch("ou=Groups,ou=MSK,dc=itdraft,dc=ru", ldap.SCOPE_SUBTREE, "(objectClass=group)")

AUTH_LDAP_GROUP_TYPE = NestedGroupOfNamesType(name_attr="cn")


AUTH_LDAP_ALWAYS_UPDATE_USER = True

#AUTH_LDAP_MIRROR_GROUPS = True

AUTH_LDAP_USER_FLAGS_BY_GROUP = {
    "is_active": "cn=00.gruop.001,ou=Groups,ou=MSK,dc=itdraft,dc=ru",
    "is_staff": "cn=00.gruop.001,ou=Groups,ou=MSK,dc=itdraft,dc=ru",
    "is_superuser": "cn=00.gruop.001,ou=Groups,ou=MSK,dc=itdraft,dc=ru"
}

AUTH_LDAP_FIND_GROUP_PERMS = True

AUTH_LDAP_CACHE_TIMEOUT = 3600

AUTHENTICATION_BACKENDS = (
    "django_auth_ldap.backend.LDAPBackend",
    "django.contrib.auth.backends.ModelBackend",
)
```

## Заключение

После внесения изменений не забываем перезагрузить Netbox

```sh
$ sudo systemctl restart netbox netbox-rq
```
