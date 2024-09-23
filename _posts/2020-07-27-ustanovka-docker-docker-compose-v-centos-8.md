---
title: "Установка Docker и Docker Compose в CentOS 8"
date: "2020-07-27"
categories: 
  - DevOps
tags: 
  - "centos"
  - "docker"
  - "docker-compose"
image:
  path: /commons/DK0fg8CU8AAfxEf.jpg
  alt: "Установка Docker и Docker Compose в CentOS 8"
---

> **Docker Compose** - это инструмент для определения и запуска многоконтейнерных приложений на основе Docker. Он позволяет описать конфигурацию и зависимости между контейнерами в виде файла YAML (YAML Ain’t Markup Language) и запустить их с помощью одной команды. Он обеспечивает упрощение управления сложными приложениями, состоящими из множества контейнеров, и позволяет разработчикам фокусироваться на написании кода, а не на настройке окружения.
{: .prompt-tip }

При установке Docker и Docker Compose в CentOS 8 есть небольшие различия, по сравнению с CentOS 7

## Установка Docker

Устанавливаем необходимые пакеты

```sh
$ sudo dnf -y install -y yum-utils device-mapper-persistent-data lvm2
```

Добавляем репозиторий Docker CE (Community Edition)

```sh
$ sudo dnf config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo
```

Проверяем

```sh
$ dnf list docker-ce
[...]
Available Packages
docker-ce.x86_64                3:19.03.12-3.el7                docker-ce-stable
```

Устанавливаем Docker CE

```sh
$ sudo dnf -y install docker-ce --nobest
```

Добавляем нашего пользователя, под которым настраиваем ОС, в группу `docker`

```sh
$ sudo usermod -aG docker $(whoami)
```

Применяем изменения к группам

```sh
$ newgrp docker
```

Добавляем сервис в автозагрузку и запускаем его

```sh
$ sudo systemctl enable --now docker
```

Проверяем

```sh
$ docker -v
Docker version 19.03.12, build 48a66213fe
```

## Установка Docker Compose

Скачиваем Docker Compose в каталог `/usr/local/bin/`

```sh
$ sudo curl -L "https://github.com/docker/compose/releases/download/1.26.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

Делаем файл исполняемым и создаем симлинк

```sh
$ sudo chmod +x /usr/local/bin/docker-compose
$ sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```

Проверяем

```sh
$ docker-compose -v
docker-compose version 1.26.2, build eefe0d31
```

## Настраиваем Firewall

Для того, что бы контейнеры могли общаться с внешней сетью, используя IP-адрес хоста вместо собственных IP-адресов контейнеров, добавим правило `add-masquerade` в Firewall

```sh
$ sudo firewall-cmd --zone=public --add-masquerade --permanent
$ sudo firewall-cmd --reload
```
