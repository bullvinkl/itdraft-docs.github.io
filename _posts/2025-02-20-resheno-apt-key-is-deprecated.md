---
layout: post
title: "[Решено] Аpt-key is deprecated"
date: "2025-02-20"
categories:
  - Administration
tags:
  - apt-key
  - gpg
image:
  path: /commons/stress-min-1.png
  alt: "Аpt-key is deprecated"
---

> **APT-KEY** — это утилита, предназначенная для добавления ключей от репозиториев в систему. Ключи защищают репозитории от возможности подделки пакета, что гарантирует, что пакеты действительно пришли от их официальных разработчиков и не были модифицированы третьими сторонами.
{: .prompt-tip }

- В продолжении к предыдущей статье [Key is stored in legacy trusted.gpg]({% post_url 2025-02-18-resheno-nginx-key-is-stored-in-legacy-trusted-gpg-keyring %}){:target="_blank"}.

В современный Debian-Like дистрибутивах появляется сообщение
> Warning: apt-key is deprecated. Manage keyring files in trusted.gpg.d instead (see apt-key(8)).
{: .prompt-warning }

Утилита `apt-key` устарела, но при этом ключи все еще добавляются в `/etc/apt/trusted.gpg`.

Избавляемся от предупреждения

## Безопасный способ

Вместо `apt-key` пользуемся `gpg`. Устанавливаем утилиту `gnupg2`
```bash
sudo apt -y install gnupg2
```

Примеры добавления ключей некоторых репозиториев
```bash
# Nginx
wget -q -O - https://nginx.org/keys/nginx_signing.key | sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/nginx.gpg

# PostgreSQL
wget -q -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/postgresql.gpg

# Zabbix
wget -q -O - https://repo.zabbix.com/zabbix-official-repo.key | sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/zabbix.gpg
```

## Не безопасный способ

Если вы доверяете репозиторию (к примеру это ваш локальное зеркало), добавлем ключ `trusted=yes` в наш репозиторий, который ругается на ключ
```bash
sudo nano /etc/apt/sources.list.d/nginx.list

deb [trusted=yes] https://repo.itdraft.ru/zabbix/7.2/debian bookworm main
```

Таким образом мы вовсе отключаем проверку ключа репозитория
