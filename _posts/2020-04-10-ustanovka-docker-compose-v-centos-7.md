---
title: "Установка Docker Compose в Centos 7"
date: "2020-04-10"
categories: 
  - Linux
  - Docker
tags: 
  - "centos"
  - "docker"
  - "docker-compose"
image:
  path: /commons/DK0fg8CU8AAfxEf.jpg
  alt: "Установка Docker Compose"
---

> **Docker Compose** - пакетный менеджер позволяющий описывать необходимую структуру в одном файле (конфиге).

## Установка Docker

Устанавливаем необходимые пакеты

```sh
$ sudo yum install -y yum-utils device-mapper-persistent-data lvm2
```

Добавляем репозиторий docker-ce

```sh
$ sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

Устанавливаем Docker-CE

```sh
$ sudo yum install -y docker-ce
```

Добавляем нашего пользователя, под которым настраиваем ОС, в группу Docker

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

## Установка Docker Compose

Добавляем репозиторий EPEL

```sh
$ sudo yum install -y epel-release
```

Устанавливаем Python-pip

```sh
$ sudo yum install -y python-pip python-devel gcc
$ sudo yum install -y python3-pip
```

Устанавливаем Docker Compose

```sh
$ sudo pip3 install docker-compose
```

Делаем симлинк на файл docker-compose

```sh
$ sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```

Обновляем утилиту pip

```sh
$ sudo pip install --upgrade pip
```

Обновляем Python

```sh
$ sudo yum upgrade python*
```

Проверяем

```sh
$ sudo docker-compose version
```

## Установка Docker Compose, способ №2

Скачиваем docker-compose в каталог `/usr/local/bin/`

```sh
$ sudo curl -L "https://github.com/docker/compose/releases/download/1.25.3/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

Делаем файл исполняемым и создаем сим линк

```sh
$ sudo chmod +x /usr/local/bin/docker-compose
$ sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```