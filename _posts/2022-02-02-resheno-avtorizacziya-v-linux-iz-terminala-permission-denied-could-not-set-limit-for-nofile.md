---
title: "[Решено] Авторизация в Linux из терминала - Permission denied. Could not set limit for 'nofile'"
date: "2022-02-02"
categories: 
  - Linux
tags: 
  - "centos"
  - "limits"
  - "linux"
  - "nofile"
  - "permission"
image:
  path: /commons/no-release-repo.jpg
  alt: "Permission denied. Could not set limit for 'nofile'"
---

> **limits.conf** - конфигурационный файл для `pam_limits.so` модуля. Он определяет ulimit лимиты для пользователей и групп. В Linux есть системные вызовы: `getrlimit()` и `setrlimit()` для получения и установления лимитов на системные ресурсы. Конфигурация по-умолчанию лежит в `/etc/security/limits.conf`

При попытке авторизации из консоли виртуализации любым пользователей появляется сообщение: Permission denied, и далее идет срока ввода логина, т.е. следующая попытка авторизации.  
При попытке авторизации через ssh-клиент (putty), окно программы закрывается.

Загружаем ОС в однопользовательском режиме. Для этого во время загрузки ОС при выборе ядра жмем `е` и меняем `ro` на `rw init=/sysroot/bin/sh ` 
Для загрузки нажимаем `Ctrl+X`

Далее выполняем команду

```sh
# chroot /sysroot
```

Сама ошибка отображается в лог-файле /var/log/secure (для Centos)

```sh
...
PAM pam_open_sessiob(): Permission denied
...
Could not set limit for 'nofile': Operation not permitted
```

Смотрим установленный лимит на максимально количество открытых файлов

```sh
# cat /etc/security/limits.conf
```

Допустимый диапазон значений для nofile: 1-1048576

У меня он был выше разрешенного

После внесения изменений выполняем команду, чтобы включить перемаркировку SELinux, и изменения применились

```sh
# touch /.autorelabel
```

Перезагружаем систему

```sh
# reboot -f
```