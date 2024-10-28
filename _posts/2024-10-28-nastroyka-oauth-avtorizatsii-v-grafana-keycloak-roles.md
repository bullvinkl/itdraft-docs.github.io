---
layout: post
title: "Настройка OAUTH авторизации в Grafana (Keycloak, Roles)"
date: "2024-10-28"
categories:
  - Monitoring-System
tags:
  - grafana
  - keycloak
image:
  path: /commons/1_JOoga42iuLkWxj_tKIzYYg.webp
  alt: "Настройка OAUTH авторизации в Grafana (Keycloak, Roles)"
---

> **Grafana** - это свободная, открытое программное обеспечение для мониторинга и визуализации данных, ориентированная на IT-инфраструктуру. Она собирает данные из различных источников, таких как time series databases (например, InfluxDB), системы мониторинга (например, Prometheus, Graphite), SIEM-системы (например, Elasticsearch, Splunk), и отображает их в удобном для пользователя формате.
{: .prompt-tip }

## Настройка Keycloak

- [Установка Keycloak + PostgeSQL в Linux]({% post_url 2022-08-29-ustanovka-keycloak-i-postgesql-v-linux-centos-rocky-debian %}){:target="_blank"} была рассмотрена в одной из предыдущих статей.
- Список статей из категории [Keycloak](/tags/keycloak/){:target="_blank"}

В Web интерфейсе Keycloak создаем нового клиента

![Images](/assets/img/posts/2024/10/28/grafana-kc1.png){: w="300" }
_Настройки Keycloak - Client_

- Client ID: `grafana`
- Name: `grafana`
- Root URL: `https://grafana.itdraft.ru`
- Home URL: `https://grafana.itdraft.ru`
- Valid redirect URI: `https://grafana.itdraft.ru/login/generic_oauth/*`
- Web origins: `https://grafana.itdraft.ru`

![Images](/assets/img/posts/2024/10/28/grafana-kc2.png){: w="300" }
_Настройки Keycloak - Client_

- Client authentication: `On`
- Authentication flow: 
  - Standard flow: `On`
  - Direct access grants: `On`

Копируем `Client secret` в настройках клиента `grafana` (во вкладке `Credentials`)

Переходим во вкладку `Roles`, создаем 2 роли

![Images](/assets/img/posts/2024/10/28/grafana-kc3.png){: w="300" }
_Настройки Keycloak - Roles_

- grafana-admin
- grafana-editor

Создаем мэппер (mappers). Для этого переходим во вкладку `Client scopes` и проваливаемся по ссылке `grafana-dedicated` (Dedicated scope and mappers for this client)

![Images](/assets/img/posts/2024/10/28/grafana-kc4.png){: w="300" }
_Настройки Keycloak - Mappers_

Добавляем мэппер `roles`

![Images](/assets/img/posts/2024/10/28/grafana-kc5.png){: w="300" }
_Настройки Keycloak - Mappers_

- Mapper type: `User Client Role`
- Name: `roles`
- Client ID: `grafana`
- Multivalued: `On`
- Token Claim Name: `resource_access.${client_id}.roles`
- Claim JSON Type: `String`
- Add to ID token: `On`
- Add to access token: `On`
- Add to userinfo: `On`

> Для mapper type: `User Client Role` роль берется из настроек клиента (вкладка `Roles`).
>
> Для mapper type: `User Realm Role` роль берется из глобальных настроек (пункт меню `Realm roles`).
{: .prompt-info }

В Web интерфейсе Keycloak переходим в раздел Groups и создаем 2 группы:
- gr-admin
- gr-editor

Для каждой группы добавляем пользователей во вкладке Members

![Images](/assets/img/posts/2024/10/28/grafana-kc6.png){: w="300" }
_Настройки Keycloak - Members_

К каждой группе привязываем соответствующую роль во вкладке Role mapping

![Images](/assets/img/posts/2024/10/28/grafana-kc7.png){: w="300" }
_Настройки Keycloak - Assign role_

![Images](/assets/img/posts/2024/10/28/grafana-kc8.png){: w="300" }
_Настройки Keycloak - Filter by client_

- Assign role > Filter by client

Настройка Keycloak завершена

## Настройка Grafana

Редактируем файл `grafana.ini` область `[auth.generic_oauth]`

```bash
$ sudo nano /etc/grafana/grafana.ini
...
[auth.generic_oauth]
enabled = true
name = Keycloak_oauth
allow_sign_up = true
client_id = grafana
client_secret = %CLIENT_SECRET%    # From Keycloak
scopes = openid email profile offline_access roles
email_attribute_path = email
login_attribute_path = username
name_attribute_path = full_name
auth_url = https://keycloak.itdraft.ru/realms/itdraft/protocol/openid-connect/auth
token_url = https://keycloak.itdraft.ru/realms/itdraft/protocol/openid-connect/token
api_url = https://keycloak.itdraft.ru/realms/itdraft/protocol/openid-connect/userinfo
role_attribute_path = contains(resource_access.grafana.roles[*], 'admin') && 'Admin' || contains(resource_access.grafana.roles[*], 'editor') && 'Editor' || 'Viewer'
```

Перезапускаем Grafana

```bash
$ sudo systemctl restart grafana-server
```

Авторизуемся в Grafana с помощью Keyclaok, проверяем роль в профайле

![Images](/assets/img/posts/2024/10/28/grafana.png){: w="300" }
_Grafana - Profile_