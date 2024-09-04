---
title: "Монтируем WebDAV в CentOS 7 на примере Яндекс.Диск"
date: "2019-03-21"
categories: 
  - Linux
tags: 
  - "centos"
  - "davfs"
  - "mount"
  - "webdav"
image:
  path: /commons/programming-flat-60074276.jpg
  alt: "Монтируем WebDAV в CentOS"
---

> **WebDAV** (Web Distributed Authoring and Versioning) или просто DAV — набор расширений и дополнений к протоколу HTTP, поддерживающих совместную работу пользователей над редактированием файлов и управление файлами на удаленных веб-серверах. В качестве миссии рабочей группы по созданию DAV было заявлено: "разработка дополнений к протоколу HTTP, обеспечивающих свободное взаимодействие инструментов распределенной разработки веб-страниц, в соответствии с потребностями работы пользователей". Однако в процессе эксплуатации DAV нашёл себе ряд других применений, выходящих за первоначально принятые рамки коллективной работы над веб-документами. Сегодня DAV применяется в качестве сетевой файловой системы, эффективной для работы в Интернете и способной обрабатывать файлы целиком, поддерживая хорошую производительность работы в условиях окружения с высокой временной задержкой передачи информации.

Обновляем операционную систему, добавляем репозиторий EPEL и устанавливаем `davfs`

```sh
$ sudo yum update
$ sudo yum install epel-release
$ sudo yum install davfs2
```

Добавляем данные для авторизации в `Яндекс.Диске`

```sh
$ sudo nano /etc/davfs2/secrets
...
# Examples
# /home/otto/foo                otto          g3H\"x\ 7z\\
# /media/dav/bar                otto          geheim
# Old style
# "http://foo.bar/my documents" otto          "geh # heim"
# https://foo.bar:333/dav       otto          geh\ \#\ heim
https://webdav.yandex.ru        %user%        %password%
...
```

Проверяем

```sh
$ sudo mkdir /mnt/yandex
$ sudo mount -t davfs https://webdav.yandex.ru /mnt/yandex
/sbin/mount.davfs: Warning: can't write entry into mtab, but will mount the file system anyway
```