---
title: "Установка ElasticSearch в Centos 8"
date: "2020-11-09"
categories: 
  - Logging-System
  - Database-System
  - DevOps
tags: 
  - "centos"
  - "elasticsearch"
  - "elk"
image:
  path: /commons/1382848_75d8_2.jpg
  alt: "Установка ElasticSearch"
---

> **Elasticsearch** – это одна из самых популярных поисковых систем в области Big Data, масштабируемое нереляционное хранилище данных с открытым исходным кодом, аналитическая NoSQL-СУБД с широким набором функций полнотекстового поиска.
{: .prompt-tip }

## Установка ElasticSearch из репозитория

Импортируем PGP Key для дальнейшего добавления репозитория ElasticSearch

```sh
$ sudo rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch
```

Добавляем репозиторий

```sh
$ sudo nano /etc/yum.repos.d/elasticsearch.repo
[elasticsearch]
name=Elasticsearch repository for 7.x packages
baseurl=https://artifacts.elastic.co/packages/7.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
```

Устанавливаем ElasticSearch

```sh
$ sudo dnf -y install elasticsearch
```

Добавляем службу ElasticSearch в автозагрузку и запускаем ее

```sh
$ sudo systemctl enable --now elasticsearch
```

Проверяем, запустилась ли служба

```sh
$ systemctl status elasticsearch
```

## Установка ElasticSearch из RPM-пакета

Скачиваем пакет ElasticSearch и устанавливаем его

```sh
$ wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.9.3-x86_64.rpm -P /tmp
$ cd /tmp
$ sudo dnf -y localinstall elasticsearch-7.9.3-x86_64.rpm
```

Добавляем службу ElasticSearch в автозагрузку и запускаем ее

```sh
$ sudo systemctl enable --now elasticsearch
```

Проверяем, запустилась ли служба

```sh
$ systemctl status elasticsearch
```

Проверяем порт

```sh
$ ss -tulnp | grep 9200
```

Тестируем

```sh
$ curl -X GET "localhost:9200/"
```

Система должна отобразить весь список информации. Во второй строке вы должны увидеть, что для `cluster_name` установлено значение `elasticsearch`. Это подтверждает, что Elasticsearch работает и прослушивает порт 9200.

Смотрим сообщения, зарегистрированные службой Elasticsearch, используя команду

```sh
$ sudo journalctl -u elasticsearch
```

## Настройка ElasticSearch

Редактируем конфигурационный файл ElasticSearch

```sh
$ sudo nano /etc/elasticsearch/elasticsearch.yml
...
# Set the bind address to a specific IP (IPv4 or IPv6):
#
network.host: localhost
#
# Set a custom port for HTTP:
#
http.port: 9200
...
```

Меняем минимальный/максимальный размер оперативной памяти, выделяемый для java-машины

```sh
$ sudo nano /etc/elasticsearch/jvm.options
...
-Xms2g
-Xmx2g
...
```

Перезапускаем ElasticSearch

```sh
$ sudo systemctl restart elasticsearch
```

## Настройка Firewall

Открываем порт 9200

```sh
$ sudo firewall-cmd --add-port=9200/tcp --permanent
$ sudo firewall-cmd --reload
```

## Настройка SELinux

Добавляем порт службы ElasticSearch 9200/tcp в политику SELinux

```sh
$ sudo semanage port -m -t http_port_t 9200 -p tcp
```
