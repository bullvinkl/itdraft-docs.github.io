---
title: "Установка Alertmanager с авторизацией и подключение к Prometheus в Centos 8"
date: "2020-11-03"
categories: 
  - Monitoring-System
  - DevOps
tags: 
  - "alertmanager"
  - "basic-auth"
  - "centos"
  - "nginx"
  - "node-exporter"
  - "prometheus"
  - "reverse-proxy"
image:
  path: /commons/artificial-4082314_960_720.jpg
  alt: "Установка Alertmanager"
---

> **Alertmanager** - это инструмент для обработки оповещений, который устраняет дубликаты, группирует и отправляет оповещения соответствующему получателю.
{: .prompt-tip }

## Установка Alertmanager

Добавляем пользователя

```sh
$ sudo useradd -M -s /bin/false alertmanager
```

Создаем каталоги

```sh
$ sudo mkdir /etc/alertmanager /var/lib/prometheus/alertmanager
```

Скачиваем Alertmanager в каталог `/tmp`

```sh
$ wget https://github.com/prometheus/alertmanager/releases/download/v0.21.0/alertmanager-0.21.0.linux-amd64.tar.gz -P /tmp
$ cd /tmp
```

Распаковываем и копируем в системные каталоги

```sh
$ tar -zxpvf alertmanager-0.21.0.linux-amd64.tar.gz
$ cd alertmanager-0.21.0.linux-amd64
$ sudo cp alertmanager amtool /usr/local/bin/
$ sudo cp alertmanager.yml /etc/alertmanager
$ sudo chown -R alertmanager:alertmanager /etc/alertmanager /var/lib/prometheus/alertmanager
$ sudo chown alertmanager:alertmanager /usr/local/bin/{alertmanager,amtool}
```

Создаем Systemd Unit

```sh
$ sudo nano /etc/systemd/system/alertmanager.service
[Unit]
Description=Alertmanager Service
After=network.target prometheus.service

[Service]
EnvironmentFile=-/etc/default/alertmanager
User=alertmanager
Group=alertmanager
Type=simple
ExecStart=/usr/local/bin/alertmanager \
          --config.file=/etc/alertmanager/alertmanager.yml \
          --storage.path=/var/lib/prometheus/alertmanager \
          $ALERTMANAGER_OPTS
ExecReload=/bin/kill -HUP $MAINPID
Restart=on-failure
Restart=always

[Install]
WantedBy=multi-user.target
```

Добавляем в автозагрузку, запускаем сервис, проверяем статус

```sh
$ sudo systemctl daemon-reload
$ sudo systemctl enable --now alertmanager
$ sudo systemctl status alertmanager
```

Проверяем, доступен ли порт 9093

```sh
$ ss -tunlp | grep 9093
```

## Настройка авторизации

Устанавливаем утилиту `dnf-utils`

```sh
$ sudo dnf -y install dnf-utils
```

Добавляем репозиторий Nginx

```sh
$ sudo nano /etc/yum.repos.d/nginx.repo
[nginx-stable]
name=nginx stable repo
baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=1
enabled=1
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true

[nginx-mainline]
name=nginx mainline repo
baseurl=http://nginx.org/packages/mainline/centos/$releasever/$basearch/
gpgcheck=1
enabled=0
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true
```

По умолчанию будет использоваться стабильная версия. Если нужна основная версия (`mainline`), переключаемся

```sh
$ sudo dnf config-manager --set-enabled nginx-mainline
```

Устанавливаем Nginx

```sh
$ sudo dnf -y install nginx
```

Отключаем дефолтный конфиг

```sh
$ cd /etc/nginx/conf.d/
$ sudo mv default.conf default.conf.disable
```

Создаем конфиг `alertmanager.conf`

```sh
$ sudo nano alertmanager.conf
server {
    listen       19093;
    listen       [::]:19093;
    server_name  _;

    location / {
        proxy_set_header Accept-Encoding "";
        proxy_pass http://localhost:9093/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        auth_basic "AlertManager";
        auth_basic_user_file /etc/nginx/alertmanager.htpasswd;
   }
}
```

Перезагружаем Nginx, проверяем статус

```sh
$ sudo systemctl restart nginx
$ sudo systemctl status nginx
```

Редактируем Systemd Unit Alertmanager

```sh
$ sudo nano /etc/systemd/system/alertmanager.service
[Unit]
Description=Alertmanager Service
After=network.target prometheus.service

[Service]
EnvironmentFile=-/etc/default/alertmanager
User=alertmanager
Group=alertmanager
Type=simple
ExecStart=/usr/local/bin/alertmanager \
          --config.file=/etc/alertmanager/alertmanager.yml \
          --storage.path=/var/lib/prometheus/alertmanager \
          --web.external-url=http://localhost:19093 \
          --web.route-prefix=/ \
          $ALERTMANAGER_OPTS
ExecReload=/bin/kill -HUP $MAINPID
Restart=on-failure
Restart=always

[Install]
WantedBy=multi-user.target
```

Перезапускаем сервис

```sh
$ sudo systemctl daemon-reload
$ sudo systemctl restart alertmanager
$ sudo systemctl status alertmanager
```

Генерируем пароль для авторизации

```sh
$ sudo htpasswd -c /etc/nginx/alertmanager.htpasswd myalertuser
    New password: passwor
    Re-type new password: password
    Adding password for user myalertuser
```

Открываем порт 19093

```sh
$ sudo firewall-cmd --add-port=19093/tcp --permanent
$ sudo firewall-cmd --reload
$ sudo firewall-cmd --list-all
```

## Настройка SELinux

```sh
$ cd /tmp
$ sudo grep nginx /var/log/audit/audit.log | grep denied | audit2allow -m nginxlocalconf > nginxlocalconf.te
$ sudo grep nginx /var/log/audit/audit.log | grep denied | audit2allow -M nginxlocalconf
    ******************** IMPORTANT ***********************
    To make this policy package active, execute:
    semodule -i nginxlocalconf.pp
$ sudo semodule -i nginxlocalconf.pp
```

## Интеграция с Prometheus

Создаем каталог, где будут лежать правила

```sh
$ sudo mkdir /etc/prometheus/rules.d/
```

Создадим файлы с правилами оповещения для Prometheus

```sh
$ sudo nano /etc/prometheus/rules.d/alert.rules.yml
```
```yaml
groups:
- name: Instance.rules
  rules:
  - alert: InstanceDown
    expr: up == 0
    for: 1m
    labels:
      severity: critical
    annotations:
      description: '{ {$labels.instance} } of job { {$labels.job} } has been down for more than 1 minute.'
    summary: 'Instance { {$labels.instance} } down'

- name: Endpoint.rules
  rules:
    - alert: EndpointDown
      expr: probe_success == 0
      for: 10s
      labels:
        severity: critical
      annotations:
        summary: 'Endpoint { {$labels.instance} } down' 
```

> Фигурные скобки `{ {` и `} }` следует писать без пробела между ними. Движок сайта некорректно их обрабатывает.
{: .prompt-info }

```sh
$ sudo nano /etc/prometheus/rules.d/system.rules.yml
```
```yaml
groups:
# Диск забит
- name: Disk-usage
  rules:
  - alert: 'Low data disk space'
    expr: ceil(((node_filesystem_size_bytes{mountpoint!="/boot"} - node_filesystem_free_bytes{mountpoint!="/boot"}) / node_filesystem_size_bytes{mountpoint!="/boot"} * 100)) > 95
    labels:
      severity: "critical"
    annotations:
      title: "Disk Usage"
      description: 'Partition : { {$labels.mountpoint} }'
      summary: "Disk usage is { {humanize $value} }% "
      host: " { {$labels.instance} } " 

# Память забита
- name: Memory-usage
  rules:
  - alert: 'High memory usage'
    expr: ceil((((node_memory_MemTotal_bytes - node_memory_MemFree_bytes - node_memory_Buffers_bytes - node_memory_Cached_bytes) / node_memory_MemTotal_bytes) * 100)) > 80
    labels:
      severity: "critical"
    annotations:
      title: "Memory Usage"
      description: 'Memory usage threshold set to 80%.'
      summary: "Memory usage is { {humanize $value} }%"
      host: "{ {$labels.instance} }"

# Процессор загружен
- name: CPU-Hight-Load
  rules: 
  - alert: HighSystemLoad
    expr: systemload_average > 90
    for: 5s
    labels:
      severity: "critical"
    annotations:
      title: "Memory Usage"
      summary: "High system load: { {$value | printf \"%.2f\"} }%"
      host: "{ {$labels.instance} }" 
```

> Фигурные скобки `{ {` и `} }` следует писать без пробела между ними. Движок сайта некорректно их обрабатывает.
{: .prompt-info }

```sh
$ sudo nano /etc/prometheus/rules.d/services.rules.yml
```
```yaml
groups:
- name: services.rules
  rules:
    - alert: services
      expr: node_systemd_unit_state{state="active"} == 0
      for: 1s
      annotations:
        summary: "Instance { {$labels.instance} } is down"
description: "{ {$labels.instance} } of job { {$labels.job} } is down." 
```

> Фигурные скобки `{ {` и `} }` следует писать без пробела между ними. Движок сайта некорректно их обрабатывает.
{: .prompt-info }

Добавляем список правил в Prometheus

```sh
$ sudo nano /etc/prometheus/prometheus.yml
```
```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s
# Alertmanager configuration
alerting:
  alertmanagers:
  - static_configs:
    - targets:
      - localhost:9093 

rule_files:
# - "alert.rules.yml"
# - "system.rules.yml"
# - "web.rules.yml"
# - "services.rules.yml"
  - "/etc/prometheus/rules.d/*.rules.yml" 

scrape_configs:
...
```

Меняем права

```sh
$ sudo chown -R prometheus. /etc/prometheus/
```

Проверяем правила на ошибки

```sh
$ sudo /usr/local/bin/promtool check rules /etc/prometheus/alert.rules.yml
Checking /etc/prometheus/alert.rules.yml
  SUCCESS: 1 rules found
```

Перезапускаем Prometheus

```sh
$ sudo systemctl restart prometheus
$ sudo systemctl status prometheus
```

Создаем файл с настройками оповещений Alertmanager

```sh
$ sudo nano /etc/alertmanager/alertmanager.yml
```
```yaml
global:
  resolve_timeout: 5m
#  smtp_smarthost: 'smtp.gmail.com:587'
#  smtp_from: 'example@gmail.com'
#  smtp_auth_username: 'example@gmail.com'
#  smtp_auth_identity: 'example@gmail.com'
#  smtp_auth_password: 'passwd'

route:
  group_by: [Alertname]
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 1h
  # default - send 'info' to email only
  receiver: default

receivers:
- name: default
  email_configs:
  - to: admin@itdraft.ru
    send_resolved: true
    from: example@gmail.com
    smarthost: smtp.gmail.com:587
    auth_username: "example@gmail.com"
    auth_identity: "example@gmail.com"
    auth_password: "passwd" 
```

Перезапускаем Alertmanager

```sh
$ sudo systemctl restart alertmanager
$ sudo systemctl status alertmanager
```

> AlertManager с smtp работает только по портам 25, 587
{: .prompt-info }

## Продолжение настройки Node Exporter

Что бы можно было мониторить запущенные сервисы, редактируем Systemd Unit для Node Exporter

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
ExecStart=/usr/local/bin/node_exporter \
     --collector.systemd \
     --collector.systemd.unit-whitelist="(sshd|chronyd|nginx).service" \
     --collector.filesystem.ignored-mount-points=^/(sys|proc|dev|host|etc)($$|/) \
     --web.config=/opt/node_exporter/web.yml
[Install]
WantedBy=multi-user.target
```

Перезапускаем сервис

```sh
$ sudo systemctl daemon-reload
$ sudo systemctl restart node_exporter
```
