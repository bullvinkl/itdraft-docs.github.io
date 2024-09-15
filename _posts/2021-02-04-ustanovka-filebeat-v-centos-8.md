---
title: "Установка Filebeat в Centos 8"
date: "2021-02-04"
categories: 
  - Logging-System
  - DevOps
tags: 
  - "filebeat"
  - "kibana"
image:
  path: /commons/1.png
  alt: "Установка Filebeat"
---

> **Filebeat** — клиент для передачи логов в logstash. Работает в совокупности с ELK стеком
{: .prompt-tip }

## Установка Filebeat из репозитория

Импортируем PGP Key для дальнейшего добавления репозитория ElasticSearch

```sh
$ rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch
```

Добавляем репозиторий

```sh
$ sudo nano /etc/yum.repos.d/kibana.repo
[elasticsearch]
name=Elasticsearch repository for 7.x packages
baseurl=https://artifacts.elastic.co/packages/7.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
```

Устанавливаем Filebeat

```sh
$ sudo dnf -y install filebeat
```

Добавим модуль, который будет проверять локальные системные журналы

```sh
$ sudo filebeat modules enable system
```

Запускаем настройку Filebeat

```sh
$ sudo filebeat setup
```

Система выполнит некоторую работу, просканирует вашу систему и подключится к панели управления Kibana

Добавляем службу Filebeat в автозагрузку и запускаем ее

```sh
$ sudo systemctl enable --now filebeat
```

Проверяем, запустилась ли служба

```sh
$ systemctl status filebeat
```

## Установка Filebeat из RPM-пакета

Скачиваем пакет Filebeat и устанавливаем его

```sh
$ wget https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.9.3-x86_64.rpm -P /tmp
$ cd /tmp
$ sudo dnf -y localinstall filebeat-7.9.3-x86_64.rpm
```

Добавляем службу Filebeat в автозагрузку и запускаем ее

```sh
$ sudo systemctl enable --now filebeat
```

Проверяем, запустилась ли служба

```sh
$ sudo systemctl status filebeat
```
