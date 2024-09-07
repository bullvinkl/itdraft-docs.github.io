---
layout: post
title: "[Решено] Обновить Netbox 3.7 до 4.0 в Rocky Linux 9"
date: "2024-05-15 09:55 +0300"
categories: 
  - Linux
  - Netbox
tags: 
  - netbox
  - python
  - rocky-linux
image:
  path: /commons/transformers-min.png
  alt: "Обновить Netbox 3.7 до 4.0 в Rocky Linux 9"
---

> **Netbox** — веб приложение с открытым исходным кодом, разработанное для управления и документирования компьютерных сетей. Изначально Netbox придуман командой сетевых инженеров DigitalOcean специально для системных администраторов.
{: .prompt-tip }

В операционной системе Rocky Linux по умолчанию установлен `Python 3.9`. Для установки `Netbox v.4` требует `Python 3.10`

В одной из прошлых статей я рассмотрел вариант установки [Python 3.12 в Rocky Linux 9]({% post_url 2024-01-18-resheno-ustanovka-python-3-12-v-rocky-linux-9-almalinux-i-naznachaem-ego-dlja-ispolzovanija-po-umolchaniju %})

## Подготовка к обновления

Перед обновление обязательно делаем [бэкап]({% post_url 2023-12-20-resheno-netbox-backup-restore-upgrade %})

Обновляем ОС до релиза и перезагружаемся
```sh
$ sudo dnf -y update
$ sudo reboot
```

Скачиваем архив, распаковываем его, меняем владельца
```sh
$ cd /opt
$ sudo wget https://github.com/netbox-community/netbox/archive/refs/tags/v4.0.1.tar.gz
$ sudo tar -xvf v4.0.1.tar.gz
$ sudo chown -R netbox:netbox netbox-4.0.1
```

Переносим конфиги
```sh
$ sudo cp /opt/netbox-3.7.7/netbox/netbox/configuration.py /opt/netbox-4.0.1/netbox/netbox/configuration.py
$ sudo cp /opt/netbox-3.7.7/local_requirements.txt /opt/netbox-4.0.1/local_requirements.txt
$ sudo -u netbox cp /opt/netbox-4.0.1/contrib/gunicorn.py /opt/netbox-4.0.1/gunicorn.py
```

Пересоздаем симлинк
```sh
$ sudo rm netbox
$ sudo ln -s netbox-4.0.1 netbox
$ sudo chown -h netbox:netbox netbox
```

## Установка необходимых пакетов

Без установки необходимых пакетов Netbox не запустится, т.к. некоторые компоненты python требуют дополнительных библиотек
```sh
$ sudo dnf --enablerepo=crb install perl-IPC-Run
$ sudo dnf install libpq
$ sudo dnf groupinstall "Development Tools"
$ sudo dnf install libpq-devel
$ sudo dnf install python3-devel
$ sudo dnf install postgresql15-devel
```

## Ошибка обновления

Во время обновления Netbox появлялась следующая ошибка
```
Collecting psycopg-c==3.1.18 (from psycopg[c,pool]==3.1.18->-r requirements.txt (line 28))
  Using cached psycopg-c-3.1.18.tar.gz (561 kB)
  Installing build dependencies ... done
  Getting requirements to build wheel ... done
  Preparing metadata (pyproject.toml) ... error
  error: subprocess-exited-with-error

  × Preparing metadata (pyproject.toml) did not run successfully.
  │ exit code: 1
  ╰─> [8 lines of output]
      running dist_info
      creating /tmp/pip-modern-metadata-xyoiy4dk/psycopg_c.egg-info
      writing /tmp/pip-modern-metadata-xyoiy4dk/psycopg_c.egg-info/PKG-INFO
      writing dependency_links to /tmp/pip-modern-metadata-xyoiy4dk/psycopg_c.egg-info/dependency_links.txt
      writing top-level names to /tmp/pip-modern-metadata-xyoiy4dk/psycopg_c.egg-info/top_level.txt
      writing manifest file '/tmp/pip-modern-metadata-xyoiy4dk/psycopg_c.egg-info/SOURCES.txt'
      couldn't run 'pg_config' --includedir: [Errno 2] No such file or directory: 'pg_config'
      error: [Errno 2] No such file or directory: 'pg_config'
      [end of output]
```

Решение ошибки
```sh
$ sudo PATH=$PATH:/usr/pgsql-15/bin/ pip install psycopg-c==3.1.18
```

## Обновление

Переходим в каталог, запускаем обновление, перезапускаем сервисы
```sh
$ cd /opt/netbox
$ sudo ./upgrade.sh
$ sudo systemctl restart netbox netbox-rq
```
