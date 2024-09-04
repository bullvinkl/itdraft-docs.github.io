---
title: "Установка Docker в Centos 7"
date: "2019-08-12"
categories: 
  - Linux
  - Docker
tags: 
  - "centos"
  - "docker"
image:
  path: /commons/DK0fg8CU8AAfxEf.jpg
  alt: "Установка Docker"
---

> **Docker** — программное обеспечение для автоматизации развёртывания и управления приложениями в средах с поддержкой контейнеризации. Позволяет «упаковать» приложение со всем его окружением и зависимостями в контейнер, который может быть перенесён на любую Linux-систему с поддержкой cgroups в ядре, а также предоставляет среду по управлению контейнерами.

Установим репозитории EPEL и REMI

```sh
$ sudo yum -y install epel-release
$ sudo yum -y install http://rpms.remirepo.net/enterprise/remi-release-7.rpm
```

Установим программное обеспечение и обновимся

```sh
$ sudo yum -y install wget nano mc zip upzip htop yum-utils
$ sudo yum -y update
```

Установим репозиторий Docker

```sh
$ sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

Устанавливаем Docker

```sh
$ sudo yum -y install docker-ce
```

Запускаем его и добавляем в автозагрузку

```sh
$ sudo systemctl start docker
$ sudo systemctl enable docker
```

Для проверки, запустим образ hello-world

```sh
$ sudo docker run hello-world
```

Команды для работы с контейнерами будут рассмотрены в другой статье