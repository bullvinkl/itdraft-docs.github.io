---
title: "Установка Prometheus на Centos 8, Nginx Basic Auth"
date: "2020-10-28"
categories: 
  - Monitoring-System
  - DevOps
tags: 
  - centos
  - nginx
  - prometheus
  - reverse-proxy
  - selinux
image:
  path: /commons/986254_f32a_2.jpg
  alt: "Установка Prometheus на Centos 8, Nginx Basic Auth"
---

> **Prometheus** - это бесплатное программное приложение, используемое для мониторинга событий и оповещения. Он записывает метрики в реальном времени в базу данных временных рядов, построенную с использованием модели HTTP-запроса, с гибкими запросами и оповещениями в режиме реального времени.
{: .prompt-tip }

## Установка Prometheus

Добавляем системного пользователя `prometheus`
```sh
$ sudo useradd -M -s /bin/false prometheus
```

Создаем необходимые каталоги для Prometheus
```sh
$ sudo mkdir /etc/prometheus /var/lib/prometheus
$ sudo chown prometheus /var/lib/prometheus/
```

Скачиваем последнюю версию Prometheus в каталог `/tmp`
```sh
$ wget https://github.com/prometheus/prometheus/releases/download/v2.19.3/prometheus-2.19.3.linux-amd64.tar.gz -P /tmp
$ cd /tmp
```

Распаковываем
```sh
$ sudo dnf install tar
$ tar xvzf prometheus-2.19.3.linux-amd64.tar.gz
```

Устанавливаем Prometheus
```sh
$ cd prometheus-2.19.3.linux-amd64
$ sudo cp prometheus  /usr/local/bin
$ sudo cp promtool  /usr/local/bin
$ sudo cp prometheus.yml /etc/prometheus/
```

Правим конфигурационный файл `prometheus.yml`
```sh
$ sudo nano /etc/prometheus/prometheus.yml
# my global config
global:
  scrape_interval:     15s
  evaluation_interval: 15s

# Alertmanager configuration
alerting:
  alertmanagers:
  - static_configs:
    - targets:
      # - alertmanager:9093 
# Load rules once and periodically evaluate them according to the global 'evalu$
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"
# В дальнейшем будем пользоваться схемой: новое правило - новый файл
# rule_files:
#   - /etc/prometheus/rules.d/*.rules.yml

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
    - targets: ['localhost:9090'] 

# В дальнейшем добавим авторизацию в node_exporter
  - job_name: 'node'
#    basic_auth:
#      username: prometheus
#      password: password
    static_configs:
    - targets: ['localhost:9100'] 
```

Открываем порт 19090. Мы используем не стандартный порт Prometheus, ч то бы в дальнейшем через Nginx добавить Basic Auth
```sh
$ sudo firewall-cmd --add-port=19090/tcp --permanent
$ sudo firewall-cmd --reload
```

Создаем System Unit
```sh
$ sudo nano /etc/systemd/system/prometheus.service
[Unit]
Description=Prometheus Time Series Collection and Processing Server
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
    --config.file /etc/prometheus/prometheus.yml \
    --storage.tsdb.path /var/lib/prometheus/ \
    --web.console.templates=/etc/prometheus/consoles \
    --web.console.libraries=/etc/prometheus/console_libraries \
    --web.external-url http://localhost:19090 \
    --web.route-prefix=/

[Install]
WantedBy=multi-user.target
```

Добавляем сервис а автозагрузку и запускаем его
```sh
$ sudo systemctl daemon-reload
$ sudo systemctl enable --now prometheus
```

## Установка NGINX и настройка Reverse proxy

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

По умолчанию будет использоваться стабильная версия. Если нужна основная версия - `mainline`, переключаемся
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

Создаем конфиг `prometheus.conf`
```sh
server {
    listen       19090;
    listen       [::]:19090;
    server_name  _;

location /
 { 
  proxy_set_header Accept-Encoding "";
  proxy_pass http://localhost:9090/;
  proxy_set_header Host $host;
  proxy_set_header X-Real-IP $remote_addr;
  proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  proxy_set_header X-Forwarded-Proto $scheme;
  auth_basic "Prometheus";
  auth_basic_user_file "/etc/nginx/prometheus.htpasswd";
 }
}
```

Устанавливаем утилиту `httpd-tools`
```sh
$ sudo dnf -y install httpd-tools
```

Генерируем пароль
```sh
$ sudo htpasswd -c /etc/nginx/prometheus.htpasswd mypomethuser
    New password: password
    Re-type new password: password
    Adding password for user mypomethuser
```

## Настройка SELinux

Настраиваем SELinux
```sh
$ cd /tmp
$ sudo grep nginx /var/log/audit/audit.log | grep denied | audit2allow -m nginxlocalconf > nginxlocalconf.te
$ sudo grep nginx /var/log/audit/audit.log | grep denied | audit2allow -M nginxlocalconf
   ******************** IMPORTANT ***********************   To make this policy package active, execute:   semodule -i nginxlocalconf.pp$ sudo semodule -i nginxlocalconf.pp
```

Добавляем сервис Nginx в автозагрузку, запускаем его, смотрим статус
```sh
$ sudo systemctl enable --now nginx
$ sudo systemctl status nginx
```

Готово, prometheus с авторизацией доступен по адресу `http://%your_ip%:19090`
