---
layout: post
title: "[Решено] Очистить кеш службы SSSD"
date: "2024-09-12"
categories:
  - Directory-Service
tags:
  - freeipa
  - sssd
image:
  path: /commons/629418_a5b0-1.jpg
  alt: "Очистить кеш службы sssd"
---

> **SSSD** (System Security Services Daemon) - это демон, обеспечивающий доступ к идентификационным и аутентификационным провайдерам. Кэш SSSD - это локальный хранилище, в котором хранятся результаты запросов к удаленным идентификационным провайдерам, таким как LDAP, FreeIPA или Active Directory.
{: .prompt-tip }

В SSSD есть несколько способов очистки кэша:

- Удаление файлов кэша вручную. Для этого нужно остановить демон SSSD, удалить файлы в директории `/var/lib/sss/db/` и затем запустить демон SSSD заново.
- Использование утилиты `sss_cache`. Она позволяет аннулировать конкретные записи в кэше, например, для пользователя, группы или домена.

В целом, очистка кэша SSSD - это важная операция для обеспечения синхронизации и обновления информации в системе аутентификации и авторизации.

## Удаление файлов кэша вручную

Останавливаем службу SSSD, переключаемся на `root`, чистим каталог, запускаем службу SSSD

```sh
$ sudo systemctl stop sssd
$ sudo su
# rm -f /var/lib/sss/db/*.*
# systemctl start sssd
# exit
```

> `$ sudo rm -f /var/lib/sss/db/*.*` - у меня не отрабатывает, приходится переключаться на `root` пользователя
{: .prompt-warning }

## Использование утилиты `sss_cache`

Устанавливаем утилиту `sss_cache`, чистим локальный кеш службы SSSD, перезапускаем службу SSSD

```sh
$ sudo apt -y install sssd-tools     # Debian-like
$ sudo dnf -y install sssd-common    # RHEL-like
$ sudo sss_cache -E
$ sudo systemctl restart sssd
```
