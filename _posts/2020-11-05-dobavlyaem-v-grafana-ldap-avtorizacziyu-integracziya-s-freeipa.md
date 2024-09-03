---
title: "Добавляем в Grafana LDAP-авторизацию, интеграция с FreeIPA"
date: "2020-11-05"
categories: 
  - Linux
  - Monitoring
  - FreeIPA
tags: 
  - "freeipa"
  - "grafana"
  - "ldap"
image:
  path: /commons/1051548_1085_2.jpg
  alt: "Добавляем в Grafana LDAP-авторизацию"
---

> **Grafana** - это многоплатформенное веб-приложение для аналитики и интерактивной визуализации с открытым исходным кодом. Он предоставляет диаграммы, графики и предупреждения для Интернета при подключении к поддерживаемым источникам данных, также доступна версия Grafana Enterprise с дополнительными возможностями.

Редактируем конфигурационный файл Grafana

```sh
$ sudo nano /etc/grafana/grafana.ini
############################## Auth LDAP ###################
[auth.ldap]
enabled = true
config_file = /etc/grafana/ldap.toml
allow_sign_up = true
```

Редактируем файл с настройками для LDAP-подключения

```sh
$ sudo nano /etc/grafana/ldap.toml
# To troubleshoot and get more log info enable ldap debug logging in grafana.ini
# [log]
# filters = ldap:debug

[[servers]]
# Ldap server host (specify multiple hosts space separated)
рost = "freeipa.example.local"
# Default port is 389 or 636 if use_ssl = true
port = 389
# Set to true if ldap server supports TLS
use_ssl = false
# Set to true if connect ldap server with STARTTLS pattern (create connection in insecure, then upgrade to secure connection with TLS)
start_tls = false
# set to true if you want to skip ssl cert validation
ssl_skip_verify = false
# set to the path to your root CA certificate or leave unset to use system defaults
# root_ca_cert = "/path/to/certificate.crt"
# Authentication against LDAP servers requiring client certificates
# client_cert = "/path/to/client.crt"
# client_key = "/path/to/client.key"

# Search user bind dn
 bind_dn = "uid=grafana,cn=users,cn=accounts,dc=example,dc=local"
# Search user bind password
# If the password contains # or ; you have to wrap it with triple quotes. Ex """#password;"""
bind_password = 'passwd'

# User search filter, for example "(cn=%s)" or "(sAMAccountName=%s)" or "(uid=%s)"
search_filter = "(uid=%s)"

# An array of base dns to search through
search_base_dns = ["cn=users,cn=accounts,dc=example,dc=local"]

### For Posix or LDAP setups that does not support member_of attribute you can define the below settings
## Please check grafana LDAP docs for examples
# group_search_filter = "(&(objectClass=posixGroup)(memberUid=%s))"
# group_search_base_dns = ["ou=groups,dc=grafana,dc=org"]
# group_search_filter_user_attribute = "uid"

# Specify names of the ldap attributes your ldap uses
[servers.attributes]
name = "givenName"
# name = "displayName"
surname = "sn"
username = "uid"
member_of = "memberOf"
email =  "mail"

# Map ldap groups to grafana org roles
[[servers.group_mappings]]
group_dn = "cn=grafana-admins,cn=groups,cn=accounts,dc=example,dc=local"
org_role = "Admin"
# To make user an instance admin  (Grafana Admin) uncomment line below
# grafana_admin = true
# The Grafana organization database id, optional, if left out the default org (id 1) will be used
# org_id = 1

[[servers.group_mappings]]
group_dn = "cn=grafana-editors,cn=groups,cn=accounts,dc=example,dc=local"
org_role = "Editor"

[[servers.group_mappings]]
# If you want to match all (or no ldap groups) then you can use wildcard
group_dn = "*"
org_role = "Viewer"
```

Создаем во FreeIPA пользователя

```sh
login: grafana
pass: passwd
```

Создаем во FreeIPA группы

```sh
grafana-admins
grafana-editors
```

Перезапускаем Grafana

```sh
sudo systemctl restart grafana-server
```