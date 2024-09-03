---
title: "Установка Grafana в Centos 8"
date: "2020-10-30"
categories: 
  - Linux
  - Monitoring
tags: 
  - "centos"
  - "grafana"
image:
  path: /commons/913980_54f3_3.jpg
  alt: "Установка Grafana"
---

Grafana

> **Grafana** - это многоплатформенное веб-приложение для аналитики и интерактивной визуализации с открытым исходным кодом. Он предоставляет диаграммы, графики и предупреждения для Интернета при подключении к поддерживаемым источникам данных, также доступна версия Grafana Enterprise с дополнительными возможностями.

### Установка Grafana из репозитория

Добавляем репозиторий Grafana

```
$ sudo nano /etc/yum.repos.d/grafana.repo
[grafana]
name=grafana
baseurl=https://packages.grafana.com/oss/rpm
repo_gpgcheck=1
enabled=1
gpgcheck=1
gpgkey=https://packages.grafana.com/gpg.key
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt
```

Устанавливаем Grafana

```
$ sudo dng -y install grafana
```

Добавляем сервис в автозагрузку и запускаем его. Проверяем статус

```
$ sudo systemctl daemon-reload
$ sudo systemctl enable --now grafana-server
$ sudo systemctl status grafana-server
```

Открываем порт 3000

```
$ sudo firewall-cmd --add-port=3000/tcp --permanent
$ sudo firewall-cmd --reload
```

### Установка Grafana из rpm-пакета

Скачиваем финальную версию и устанавливаем

```
$ cd /tmp
$ wget https://dl.grafana.com/oss/release/grafana-7.3.4-1.x86_64.rpm
$ sudo dnf -y localinstall grafana-7.3.4-1.x86_64.rpm
```

Добавляем сервис в автозагрузку и запускаем его. Проверяем статус

```
$ sudo systemctl daemon-reload
$ sudo systemctl enable --now grafana-server
$ sudo systemctl status grafana-server
```

Запускаем браузер, переходим по адресу **http://%your-ip%:3000/** и устанавливаем пароль админа.

Далее в админке можно добавить наш Prometheus-сервер
