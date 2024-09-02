---
title: "[Решено] Обновляем Netbox"
date: "2023-02-16"
categories: 
  - Linux
  - Netbox
tags: 
  - "assetmanager"
  - "cmdb"
  - "itsm"
  - "linux"
  - "netbox"
image:
  path: /commons/no-release-repo.jpg
  alt: "Обновляем Netbox"
---

> **NetBox**, по сути, является базой данных, где можно смоделировать все объекты сети и хранить всю информацию о них.

На сервере была установлена версия 3.4.2, попробуем обновиться до релиза. Перед обновлением не забываем сделать backup / snapshot

Переходим в каталог с установленным Netbox

```sh
$ cd /opt/netbox
```

## Обновление Netbox, установленного из github репозитория

Переключаемся на ветку master и выкачиваем изменения из репозитория (в предыдущей статье мы устанавливали netbox из git-репозиторий)

```sh
$ sudo git checkout master
$ sudo git pull origin master
```

Запускаем процесс обновления

```sh
$ sudo ./upgrade.sh
```

Не забываем перезагрузить сервисы

```sh
$ sudo systemctl restart netbox netbox-rq
```

## Обновление Netbox, установленного из архива

Если изначально вы устанавливали Netbox из архива, скачиваем архив с релизом

```sh
$ cd ~
$ wget https://github.com/netbox-community/netbox/archive/refs/tags/v3.6.7.tar.gz
$ tar -xvf v3.6.7.tar.gz
```

Переименовываем каталог с устаревшей версией Netbox, переносим распакованный релиз в каталог `/opt`

```sh
$ sudo mv /opt/netbox /opt/netbox-3.4.2
$ sudo mv ./netbox-3.6.7 /opt/netbox
```

Меняем владельца каталога `/opt/netbox`

```sh
$ sudo chown -R netbox:netbox /opt/netbox
```

Копируем конфиг из старого каталога netbox в новый

```sh
$ sudo cp /opt/netbox-3.4.2/netbox/netbox/configuration.py /opt/netbox/netbox/netbox/configuration.py
```

Копируем конфиг `gunicorn.py`

```sh
sudo -u netbox cp /opt/netbox/contrib/gunicorn.py /opt/netbox/gunicorn.py
```

Переходим в каталог с Netbox и запускаем процесс обновления

```sh
$ cd /opt/netbox
$ sudo ./upgrade.sh
```

Перезагружаем сервисы

```sh
$ sudo systemctl restart netbox netbox-rq
```