---
title: "Установка Logstash в Centos 8"
date: "2020-11-11"
categories: 
  - Logging-System
  - DevOps
tags: 
  - "centos"
  - "elasticsearch"
  - "elk"
  - "logstash"
image:
  path: /commons/629418_a5b0-1.jpg
  alt: "Установка Logstash"
---

> **Logstash** — это механизм сбора данных с открытым исходным кодом с возможностями конвейерной обработки данных в реальном времени.Logstash может динамически идентифицировать данные из различных источников и нормализовать их, с помощью выбранных фильтров.
{: .prompt-tip }

## Установка Logstash из репозитория

Импортируем PGP Key для дальнейшего добавления репозитория ElasticSearch

```sh
$ rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch
```

Добавляем репозиторий ElasticSearch

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

Устанавливаем OpenJDK 8 Java

```sh
$ sudo dnf -y install java-1.8.0-openjdk
```

> Иначе в процессе установки появится ошибка:  
> could not find java; set JAVA_HOME or ensure java is in PATH  
> chmod: cannot access '/etc/default/logstash': No such file or directory  
> warning: %post(logstash-1:7.9.3-1.noarch) scriptlet failed, exit status 1
{: .prompt-warning }

Устанавливаем Logstash

```sh
$ sudo dnf -y install logstash
```

Добавляем службу Logstash в автозагрузку и запускаем ее

```sh
$ sudo systemctl enable --now logstash
```

Проверяем, запустилась ли служба

```sh
$ systemctl status logstash
```

## Устновка Logstash из RPM-пакета

Скачиваем пакет Logstash и устанавливаем его

```sh
$ wget https://artifacts.elastic.co/downloads/logstash/logstash-7.9.3.rpm -P /tmp
$ cd /tmp
$ sudo dnf -y localinstall logstash-7.9.3.rpm 
```

Добавляем службу Logstash в автозагрузку и запускаем ее

```sh
$ sudo systemctl enable --now logstash
```

Проверяем, запустилась ли служба

```sh
$ systemctl status logstash
```
