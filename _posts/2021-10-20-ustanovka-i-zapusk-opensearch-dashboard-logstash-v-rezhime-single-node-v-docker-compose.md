---
title: "Установка и запуск Opensearch / Dashboard / Logstash в режиме single-node в docker compose"
date: "2021-10-20"
categories: 
  - Linux
  - Opensearch
tags: 
  - "docker"
  - "docker-compose"
  - "elk"
  - "logstash"
  - "opensearch"
image:
  path: /commons/best-free-proxies.png
  alt: "Opensearch / Dashboard / Logstash в режиме single-node"
---

> **Opensearch**, ранее известный как Open Distro for Elasticsearch - аналог стека ELK, но с интегрированным модулем безопасности (в ELK это платная опция)
> 
> **OpenSearch** – это распределенный комплект поисковых и аналитических ресурсов с открытым исходным кодом для различных примеров использования, таких как мониторинг приложений в режиме реального времени, анализ журналов и поиск по веб-сайтам. OpenSearch предоставляет высокомасштабируемую систему для быстрого доступа и ответа на большие объемы данных с помощью интегрированного инструмента визуализации OpenSearch Dashboards, с которым пользователям проще изучать свои данные. Как Elasticsearch и Apache Solr, OpenSearch работает на базе поисковой библиотеки Apache Lucene. OpenSearch и OpenSearch Dashboards изначально созданы на основе Elasticsearch 7.10.2 и Kibana 7.10.2.

На официальном сайте приведен пример конфигурации opensearch в 2-х нодах.  
Но в неответственных системах это лишнее использование ресурсов

```sh
$ cat docker-compose.yml
version: '3.2'
services:
  opensearch:
    image: opensearchproject/opensearch:latest
    container_name: opensearch
    environment:
      - discovery.type=single-node
      - plugins.security.ssl.http.enabled=false
      - "OPENSEARCH_JAVA_OPTS=-Xms2048m -Xmx2048m" # minimum and maximum Java heap size, recommend setting both to 50% of system RAM
    ulimits:
      memlock:
        soft: -1
        hard: -1
      nofile:
        soft: 65536 # maximum number of open files for the OpenSearch user, set to at least 65536 on modern systems
        hard: 65536
    volumes:
      - opensearch-data:/usr/share/opensearch/data
      - ./config/config.yml:/usr/share/opensearch/plugins/opensearch-security/securityconfig/config.yml
      - ./config/internal_users.yml:/usr/share/opensearch/plugins/opensearch-security/securityconfig/internal_users.yml
      - ./config/roles_mapping.yml:/usr/share/opensearch/plugins/opensearch-security/securityconfig/roles_mapping.yml
    ports:
      - 9200:9200
      - 9600:9600 # required for Performance Analyzer
    networks:
      - opensearch-net

  opensearch-dashboards:
    image: opensearchproject/opensearch-dashboards:latest
    container_name: opensearch-dashboards
    ports:
      - 5601:5601
    expose:
      - "5601"
    environment:
      OPENSEARCH_HOSTS: '["http://opensearch:9200"]'
    networks:
      - opensearch-net

  logstash:
    image: opensearchproject/logstash-oss-with-opensearch-output-plugin:latest
    container_name: logstash
    volumes:
      - ./logstash/pipeline:/usr/share/logstash/pipeline:ro
    ports:
      - "5044:5044"
    environment:
      LS_JAVA_OPTS: "-Xmx256m -Xms256m"
    networks:
      - opensearch-net
    depends_on:
      - opensearch

volumes:
  opensearch-data:
    driver: local
    driver_opts:
       o: bind
       type: none
       device: /opt/data/app/opensearch/storage/opensearch-data

networks:
  opensearch-net:
```

В данном примере так же присутствует интеграция с LDAP-сервером FreeIPA

```sh
$ cat ./config/config.yml
---
_meta:
  type: "config"
  config_version: 2

config:
  dynamic:
    http:
      anonymous_auth_enabled: false
    authc:
      internal_auth:
        order: 0
        description: "HTTP basic authentication using the internal user database"
        http_enabled: true
        transport_enabled: true
        http_authenticator:
          type: basic
          challenge: true
        authentication_backend:
          type: internal
      ldap_auth:
        order: 5
        description: "Authenticate using FreeIPA"
        http_enabled: true
        transport_enabled: true
        http_authenticator:
          type: basic
          challenge: true
        authentication_backend:
          type: ldap
          config:
            enable_ssl: false
            enable_start_tls: false
            enable_ssl_client_auth: false
            verify_hostnames: true
            hosts:
            - ipa.itdraft.ru
            bind_dn: 'uid=opensearch,cn=users,cn=accounts,dc=itdraft,dc=ru'
            password: 'passwd'
            userbase: 'cn=users,cn=accounts,dc=itdraft,dc=ru'
            usersearch: '(uid={0})'
            username_attribute: 'uid'

    authz:
      ldap_roles:
        description: "Authorize using FreeIPA"
        http_enabled: true
        transport_enabled: true
        authorization_backend:
          type: ldap
          config:
            enable_ssl: false
            enable_start_tls: false
            enable_ssl_client_auth: false
            verify_hostnames: true
            hosts:
            - 00sr-freeipa-01.gge.local
            bind_dn: 'uid=opensearch,cn=users,cn=accounts,dc=itdraft,dc=ru'
            password: 'passwd'
            userbase: 'cn=users,cn=accounts,dc=itdraft,dc=ru'
            usersearch: '(uid={0})'
            username_attribute: 'uid'
            skip_users:
              - admin
              - kibanaserver
            rolebase: 'cn=opensearch,cn=groups,cn=accounts,dc=gge,dc=local'
#            rolesearch: '(memberUid={0})'
            rolesearch: '(memberUid={1})'
            userroleattribute: null
            userrolename: memberOf
#            userrolename: disabled
#            userrolename: null
            rolename: cn
            resolve_nested_roles: true
```

Пользователи из FreeIPA-группы opensearch имеют админские права

```sh
$ cat ./config/roles_mapping.yml

---

_meta:
  type: "rolesmapping"
  config_version: 2

all_access:
  reserved: false
  backend_roles:
  - "admin"
  - "Administrator"
  - "opensearch"
  description: "Maps admin to all_access"

own_index:
  reserved: false
  users:
  - "*"
  description: "Allow full access to an index named like the username"

logstash:
  reserved: false
  hidden: false
  backend_roles:
  - "logstash"
#  hosts: []
#  users: []
#  and_backend_roles: []

kibana_user:
  reserved: false
  backend_roles:
  - "kibanauser"
  - "Developers"
  description: "Maps kibanauser to kibana_user"

readall:
  reserved: false
  backend_roles:
  - "readall"
  - "Developers"

manage_snapshots:
  reserved: false
  backend_roles:
  - "snapshotrestore"
  - "Developers"

kibana_server:
  reserved: true
  users:
  - "kibanaserver"
```