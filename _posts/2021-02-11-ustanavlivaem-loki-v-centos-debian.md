---
title: "Устанавливаем Loki в CentOS / Debian"
date: "2021-02-11"
categories: 
  - Linux
tags: 
  - "grafana"
  - "loki"
  - "prometheus"
  - "promtail"
image:
  path: /commons/laptop-2.jpg
  alt: "Устанавливаем Loki"
---

> **Loki** - хранилище для логов (prometheus like), т.е. набор компонентов для полноценной системы работы с логами.
{: .prompt-tip }

Добавляем системного пользователя, от которого будет работать Loki

```sh
$ sudo useradd -r -M -s /bin/false loki
```

Скачиваем Loki

```sh
$ cd /usr/local/bin
$ sudo curl -O -L "https://github.com/grafana/loki/releases/download/v2.0.0/loki-linux-amd64.zip"
```

Распаковываем

```sh
Распаковываем
$ sudo unzip loki-linux-amd64.zip
```

Удаляем архив

```sh
$ sudo rm loki-linux-amd64.zip
```

Делам файл исполняемым

```sh
$ sudo chmod a+x "loki-linux-amd64"
```

Меняем владельца

```sh
$ sudo chown loki:loki loki-linux-amd64
```

Создаем конфигурационный файл, либо скачиваем готовый

```sh
$ wget https://raw.githubusercontent.com/grafana/loki/master/cmd/loki/loki-local-config.yaml
```

```sh
$ sudo nano config-loki.yml
auth_enabled: false

server:
  http_listen_port: 3100

ingester:
  lifecycler:
    address: 127.0.0.1
    ring:
      kvstore:
        store: inmemory
      replication_factor: 1
    final_sleep: 0s
  chunk_idle_period: 5m
  chunk_retain_period: 30s
  max_transfer_retries: 0

schema_config:
  configs:
    - from: 2018-04-15
      store: boltdb
      object_store: filesystem
      schema: v11
      index:
        prefix: index_
        period: 168h

storage_config:
  boltdb:
    directory: /tmp/loki/index

  filesystem:
    directory: /tmp/loki/chunks

limits_config:
  enforce_metric_name: false
  reject_old_samples: true
  reject_old_samples_max_age: 168h

chunk_store_config:
  max_look_back_period: 0s

table_manager:
  retention_deletes_enabled: false
  retention_period: 0s
```

> В address: 127.0.0.1 - слушать localhost  
> В address: 0.0.0.0 указываем прослушивать все доступные интерфейсы.

Меняем владельца конфига

```sh
$ sudo chown loki:loki config-loki.yml
```

Создаем Systemd Unit

```sh
$ sudo nano /etc/systemd/system/loki.service
[Unit]
Description=Loki service
After=network.target

[Service]
Type=simple
User=loki
ExecStart=/usr/local/bin/loki-linux-amd64 -config.file /usr/local/bin/config-loki.yml

[Install]
WantedBy=multi-user.target
```

Добавляем сервис в автозагрузку и стартуем

```sh
$ sudo systemctl daemon-reload
$ sudo systemctl enable --now loki
$ systemctl status loki
```

Смотрим логи сервиса

```sh
$ sudo tail -f /var/log/messages | grep loki
```

Проверяем порт 3100, запустился ли Loki

```sh
$ ss -nltup | grep 3100
```

Настраиваем Firewall, открываем порт наружу, если нужно

```sh
$ sudo firewall-cmd --add-port=3100/tcp  --permanent
$ sudo firewall-cmd --reload
```

```
http://[Your-Server-Domain-or-IP]:3100/metrics
```

Дальше останется подключить Loki к системе визуализации Grafana, и установить на сервера, с которых мы хотим снимать логи, агент - Promtail
