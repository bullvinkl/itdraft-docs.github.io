---
title: "Установка Node Exporter с авторизацией и подключение к Prometheus в Centos 8"
date: "2020-11-02"
categories: 
  - Linux
  - Monitoring
tags: 
  - "basic-auth"
  - "centos"
  - "node-exporter"
  - "prometheus"
image:
  path: /commons/0_HICLyAdNSIyT0ODU-1.jpg
  alt: "Установка Node Exporter"
---

> **Node Exporter** — это экспортер Prometheus для сбора данных о состоянии сервера с подключаемыми коллекторами метрик. Он позволяет измерять различные ресурсы машины, такие как использование памяти, диска и процессора. Написана на Go
{: .prompt-tip }

## Установка Node Exporter

Добавляем системного пользователя, от которого будет работать Node Exporter

```sh
$ sudo useradd -r -M -s /bin/false node_exporter
```

Скачиваем `node_exporter-1.0.1`

```sh
$ wget https://github.com/prometheus/node_exporter/releases/download/v1.0.1/node_exporter-1.0.1.linux-amd64.tar.gz -P /tmp
$ cd /tmp
```

Распаковываем, переносим в каталог /usr/local/bin, назначаем владельца

```sh
$ tar -zxpvf node_exporter-1.0.1.linux-amd64.tar.gz
$ cd node_exporter-1.0.1.linux-amd64
$ sudo cp node_exporter /usr/local/bin
$ sudo chown node_exporter:node_exporter /usr/local/bin/node_exporter
```

Создаем Systemd Unit

```sh
$ sudo nano /etc/systemd/system/node_exporter.service
[Unit]
Description=Prometheus Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
```

Добавляем сервис в автозагрузку, запускаем его, проверяем статус

```sh
$ sudo systemctl daemon-reload
$ sudo systemctl enable --now node_exporter
$ sudo systemctl status node_exporter
```

Проверяем порт 9100

```sh
$ sudo ss -pnltu | grep 9100
```

Открываем его наружу

```sh
$ sudo firewall-cmd --add-port=9100/tcp  --permanent
$ sudo firewall-cmd --reload
```

## Настройка авторизации

Устанавливаем утилиту httpd-tools

```sh
$ sudo dnf -y install httpd-tools
```

Генерируем пароль

```sh
$ htpasswd -nBC 10 "" | tr -d ':\n'
New password: password
Re-type new password: password
```

Создаем каталог, где будет лежать конфиг node\_exporter

```sh
$ sudo mkdir /opt/node_exporter
```

Создаем конфигурационный файл для Node Exporter

```sh
$ sudo nano /opt/node_exporter/web.yml
#Если нужно https
#tls_server_config:
#  cert_file: node_exporter.crt
#  key_file: node_exporter.key
basic_auth_users:
  myuser: $2y$10$ZNpUythMY9kLqsldfkjsljfdlskjdflksdjf527W8kBkHPT4Rkl9C
```

Меняем владельца

```sh
$ sudo chown -R node_exporter:node_exporter /opt/node_exporter
```

Редактируем Systemd Unit

```sh
$ sudo nano /etc/systemd/system/node_exporter.service
[Unit]
Description=Prometheus Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter --web.config=/opt/node_exporter/web.yml

[Install]
WantedBy=multi-user.target
```

Перезапускаем сервис

```sh
$ sudo systemctl daemon-reload
$ sudo systemctl restart node_exporter
$ sudo systemctl status node_exporter
```

Переходим на сайт `http://%your-ip%:9100/`. Должна появиться Basic Auth

## Донастраиваем Prometheus

На сервере Prometheus добавляем параметры авторизации в конфиг Prometheus

```sh
$ sudo nano /etc/prometheus/prometheus.yml
...
  - job_name: 'node'
    basic_auth:
      username: myuser
      password: password
#    scheme: https
#    tls_config:
#      ca_file: node_exporter.crt
    static_configs:
    - targets: ['localhost:9100'] 
```

где вместо `localhost` надо указать `ip-адрес` сервера, куда мы ставили node exporter

Перезапускаем Prometheus

```sh
$ sudo systemctl restart prometheus
$ sudo systemctl status prometheus
```

Проверяем через браузер

```
http://localhost:9090/targets
```
