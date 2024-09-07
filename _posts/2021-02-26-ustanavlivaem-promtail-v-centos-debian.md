---
title: "Устанавливаем Promtail в CentOS / Debian"
date: "2021-02-26"
categories: 
  - Logging-System
  - DevOps
tags: 
  - "grafana"
  - "loki"
  - "promtail"
image:
  path: /commons/virtualisationimage.png
  alt: "Устанавливаем Promtail"
---

> **Promtail** - агент, отвечающий за централизацию логов, то есть собирает логи, обрабатывает их и отправляет в Loki. В свою очередь Loki их хранит, а Grafana запрашивает данные из Loki и визуализирует их.
{: .prompt-tip }

Добавляем системного пользователя, от которого будет работать Promtail

```sh
$ sudo useradd -r -M -s /bin/false promtail
```

Скачиваем promtail

```sh
$ cd /usr/local/bin
$ sudo curl -O -L https://github.com/grafana/loki/releases/download/v2.0.0/promtail-linux-amd64.zip
```

Распаковываем

```sh
$ sudo unzip promtail-linux-amd64.zip
```

Удаляем архив

```sh
$ sudo rm promtail-linux-amd64.zip
```

Делам файл исполняемым

```sh
$ sudo chmod a+x "promtail-linux-amd64"
```

Меняем владельца

```sh
$ sudo chown promtail:promtail promtail-linux-amd64
```

Создаем конфигурационный файл, либо скачиваем готовый

```sh
$ wget https://raw.githubusercontent.com/grafana/loki/master/cmd/promtail/promtail-local-config.yaml
```

```sh
$ sudo nano config-promtail.yml
server:
  http_listen_port: 9080
  grpc_listen_port: 0

positions:
  filename: /tmp/positions.yaml

clients:
  - url: http://127.0.0.1:3100/loki/api/v1/push
#    basic_auth:
#      username: user
#      password: pass

scrape_configs:
  - job_name: system
    static_configs:
    - targets:
        - localhost
      labels:
        job: varlogs
        __path__: /var/log/*log

  - job_name: journal
    journal:
      max_age: 12h
      labels:
        job: systemd-journal
    relabel_configs:
      - source_labels: ['__journal__systemd_unit']
        target_label: 'unit'
```

Меняем владельца конфига

```sh
$ sudo chown promtail:promtail config-promtail.yml
```

Создаем Systemd Unit

```sh
$ sudo nano /etc/systemd/system/promtail.service
[Unit]
Description=Promtail service
After=network.target

[Service]
Type=simple
User=promtail
ExecStart=/usr/local/bin/promtail-linux-amd64 -config.file /usr/local/bin/config-promtail.yml

[Install]
WantedBy=multi-user.target
```

Добавляем сервис в автозагрузку и стартуем

```sh
$ sudo systemctl daemon-reload
$ sudo systemctl enable --now promtail
$ systemctl status promtail
```

Проверяем порт 9080

```sh
$ ss -nltup | grep 9080
```

Открываем его наружу

```sh
$ sudo firewall-cmd --add-port=9080/tcp  --permanent
$ sudo firewall-cmd --reload
```

Чтобы дать возможность Promtail читать системный журналы, добавляем пользователя promtail в группу systemd-journal

```sh
$ sudo usermod -aG systemd-journal promtail
```

Перезапускаем службу

```sh
$ sudo systemctl restart promtail
```
