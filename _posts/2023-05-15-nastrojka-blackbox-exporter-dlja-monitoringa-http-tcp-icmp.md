---
title: "Настройка Blackbox Exporter для мониторинга HTTP, TCP, ICMP"
date: "2023-05-15"
categories:
  - Linux
  - Monitoring
tags: 
  - blackbox-exporter
  - linux
  - prometheus
image:
  path: /commons/p9lahdjjxckbx3hsm3uqaioj4bu.png
  alt: "Настройка Blackbox Exporter для мониторинга HTTP, TCP, ICMP"
---

> **Blackbox exporter** — экспортер для Prometheus, который реализует сбор метрик внешних сервисов через HTTP, HTTPS, DNS, TCP, ICMP. Основное предназначение - проверять SSL-сертификаты и уведомлять (с помощью Alertmanager) о том, что срок действия сертификата завершается. 

- [Установка Prometheus]({% post_url 2020-10-28-ustanovka-prometheus-na-centos-8-nginx-basic-auth %}) была рассмотрена в одной из предыдущих статей

## Установка и настройка Blackbox Exporter

Добавляем пользователя и создаем каталог для конфига Blackbox Exporter
```sh
$ sudo useradd -M -s /bin/false blackbox
$ sudo mkdir /opt/blackbox
```

Скачиваем финальную версию, распаковываем её и копируем в каталог
```sh
$ wget https://github.com/prometheus/blackbox_exporter/releases/download/v0.23.0/blackbox_exporter-0.23.0.linux-amd64.tar.gz
$ tar xzf blackbox_exporter-0.23.0.linux-amd64.tar.gz
$ cd blackbox_exporter-0.23.0.linux-amd64
$ sudo cp blackbox_exporter /usr/local/bin/
```

Создаем системный юнит
```sh
$ sudo nano /etc/systemd/system/blackbox_exporter.service
[Unit]
Description=Blackbox Exporter
Documentation=https://github.com/prometheus/blackbox_exporter
Wants=network-online.target
After=network-online.target

[Service]
User=blackbox
Group=blackbox
Restart=on-failure
RestartSec=5
Type=simple
AmbientCapabilities=CAP_NET_RAW
ExecStart=/usr/local/bin/blackbox_exporter \
    --config.file="/opt/blackbox/blackbox.yml" \
    --web.listen-address="127.0.0.1:9115"
ExecReload=/bin/kill -HUP $MAINPID

[Install]
WantedBy=multi-user.target
```

Параметр `CAP_NET_RAW` добавляется, что б Blackbox Exporter корректно выполнял ICMP-проверку хостов

Перечитывам юниты
```sh
$ sudo systemctl daemon-reload
```

Создаем конфигурационный файл `blackbox.yml`
```sh
$ sudo nano /opt/blackbox/blackbox.yml
modules:
  http_2xx:
    prober: http
    timeout: 5s
    http:
      valid_status_codes: []
      method: GET
      preferred_ip_protocol: ip4
      fail_if_ssl: false
      fail_if_not_ssl: true
      tls_config:
        insecure_skip_verify: true
  http_post_2xx:
    prober: http
    http:
      method: POST
  tcp_connect:
    prober: tcp
    timeout: 5s
    tcp:
      preferred_ip_protocol: "ip4"
  ssh_banner:
    prober: tcp
    tcp:
      query_response:
      - expect: "^SSH-2.0-"
  icmp:
    prober: icmp
    timeout: 5s
    icmp:
      preferred_ip_protocol: "ip4"
      ip_protocol_fallback: false
```

Добавляем сервис в автозагрузку и запускаем его
```sh
$ sudo systemctl enable --now blackbox_exporter
$ sudo systemctl status blackbox_exporter
```

## Настройка Prometheus

Редактируем конфигурационный файл Prometheus, добавляем проверки: ICMP, tcp-портов, HTTP (в том числе и валидность ssl-сертификатов)
```sh
$ sudo nano /etc/prometheus/prometheus.yml 
...
  - job_name: 'Blackbox-ICMP'
    scrape_interval: 5m
    metrics_path: /probe
    params:
      module: [icmp]
    file_sd_configs:
      - files:
        - /etc/prometheus/targets.d/blackbox-icmp.yml
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: 127.0.0.1:9115

  - job_name: 'Blackbox-TCP'
    scrape_interval: 5m
    metrics_path: /probe
    params:
      module: [tcp_connect]
    file_sd_configs:
      - files:
        - /etc/prometheus/targets.d/blackbox-tcp.yml
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: 127.0.0.1:9115

  - job_name: 'Blackbox-HTTP'
    scrape_interval: 5m
    metrics_path: /probe
    params:
      module: [http_2xx]
    file_sd_configs:
      - files:
        - /etc/prometheus/targets.d/blackbox-http.yml
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: 127.0.0.1:9115
...
```

Создаем файл со списком хостов для ICMP-проверки
```sh
$ sudo nano /etc/prometheus/targets.d/blackbox-icmp.yml
- targets:
  - ya.ru
  - 192.168.1.1
  - 127.0.0.1
  labels:
    alias: ping
```

Создаем файл со списком хостов для проверки tcp-портов
```sh
$ sudo nano /etc/prometheus/targets.d/blackbox-tcp.yml
- targets:
  - 192.168.1.25:1947
  labels:
    alias: tcp-connect
```

Создаем файл со списком хостов для HTTP-проверки, валидности SSL-сертификатов
```sh
$ sudo nano /etc/prometheus/targets.d/blackbox-http.yml
- targets:
  - itdraft.ru
  - ya.ru
  labels:
    alias: http
```

Перезапускаем Prometheus
```sh
$ sudo systemctl restart prometheus
$ sudo systemctl status prometheus
```
