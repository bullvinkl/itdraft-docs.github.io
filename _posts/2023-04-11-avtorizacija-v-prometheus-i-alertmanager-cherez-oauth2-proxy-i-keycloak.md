---
title: "Авторизация в Prometheus и Alertmanager через OAuth2 Proxy и Keycloak"
date: "2023-04-11"
categories: 
  - Linux
  - Keycloak
tags: 
  - alertmanager
  - debian
  - keycloak
  - linux
  - nginx
  - oauth2-proxy
  - prometheus
  - rocky-linux
  - firewall
image:
  path: /commons/ethereum_2-0_3-min.png
  alt: "Авторизация в Prometheus и Alertmanager через OAuth2 Proxy и Keycloak"
---

> **OAuth2 Proxy** — это обратный прокси-сервер, который находится перед вашим приложением и обеспечивает аутентификацию OpenID Connect / OAuth 2.0 с использованием провайдеров идентификации (Google, GitHub, Keycloak и других).
{: .prompt-tip }

- Установка Keycloak была рассмотрена в [одной из предыдущей статей]({% post_url 2022-08-29-ustanovka-keycloak-i-postgesql-v-linux-centos-rocky-debian %})

Установку Prometheus, Alertmanager и OAuth2 Proxy произвожу на одной ВМ под управлением Rocky Linux 9, по этому для начала либо настраиваем SELinux, либо отключаем его (выбрал второй вариант)

```sh
$ sudo grubby --update-kernel ALL --args selinux=0
$ sudo setenforce 0
```

## Установка Prometheus

Создаем пользователя и необходимые каталоги

```sh
$ sudo useradd -M -s /bin/false prometheus
$ sudo mkdir /opt/prometheus /var/lib/prometheus
```

Скачиваем финальную верcию Prometheus, распаковываем её и переходим в каталог

```sh
$ wget https://github.com/prometheus/prometheus/releases/download/v2.42.0/prometheus-2.42.0.linux-amd64.tar.gz
$ tar xzf prometheus-2.42.0.linux-amd64.tar.gz
$ cd prometheus-2.42.0.linux-amd64
```

Перемещаем исполняемые файлы, меняем владельца на каталоги и файлы

```sh
$ sudo mv prometheus promtool /usr/local/bin/
$ sudo mv prometheus.yml /opt/prometheus/
$ sudo chown -R prometheus. /opt/prometheus /var/lib/prometheus
$ sudo chown prometheus. /usr/local/bin/{prometheus,promtool}
```

Создаем системный юнит для Prometheus

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
    --config.file /opt/prometheus/prometheus.yml \
    --storage.tsdb.path /var/lib/prometheus/ \
    --web.console.templates=/opt/prometheus/consoles \
    --web.console.libraries=/opt/prometheus/console_libraries

[Install]
WantedBy=multi-user.target
```

Перечитываем юниты, добавляем сервис а автозагрузку и запускаем его
```sh
$ sudo systemctl daemon-reload
$ sudo systemctl enable --now prometheus
$ sudo systemctl status prometheus
```

Открываем порт 9090/tcp
```sh
$ sudo firewall-cmd --permanent --zone=public --add-port=9090/tcp
$ sudo firewall-cmd --reload
```

Чуть дальше Prometheus запустим в режиме Reverse proxy, это правило удалим

## Установка Alertmanager

Создаем пользователя и необходимые каталоги
```sh
$ sudo useradd -M -s /bin/false alertmanager
$ sudo mkdir /opt/alertmanager /var/lib/alertmanager
```

Скачиваем финальную верcию Alertmanager, распаковываем её и переходим в каталог
```sh
$ wget https://github.com/prometheus/alertmanager/releases/download/v0.25.0/alertmanager-0.25.0.linux-amd64.tar.gz
$ tar xzf alertmanager-0.25.0.linux-amd64.tar.gz
$ cd alertmanager-0.25.0.linux-amd64
```

Перемещаем исполняемые файлы, меняем владельца на каталоги и файлы
```sh
$ sudo mv alertmanager amtool /usr/local/bin/
$ sudo mv alertmanager.yml /opt/alertmanager/
$ sudo chown -R alertmanager. /opt/alertmanager /var/lib/alertmanager
$ sudo chown alertmanager. /usr/local/bin/{alertmanager,amtool}
```

Создаем системный юнит для Alertmanager
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
    --config.file=/opt/alertmanager/alertmanager.yml \
    --storage.path=/var/lib/alertmanager \
    $ALERTMANAGER_OPTS
ExecReload=/bin/kill -HUP $MAINPID
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

Перечитываем юниты, добавляем сервис а автозагрузку и запускаем его
```sh
$ sudo systemctl daemon-reload
$ sudo systemctl enable --now alertmanager
$ sudo systemctl status alertmanager
```

Открываем порт `9093/tcp`

```sh
$ sudo firewall-cmd --permanent --zone=public --add-port=9093/tcp
$ sudo firewall-cmd --reload
```

Чуть дальше Alertmanager запустим в режиме Reverse proxy, это правило удалим

## Настройка Keycloak

Версия Keycloak: 21.0.0

Переходим в нужную область (realm). Создаем клиента для OAuth2 Proxy

![](/assets/img/posts/2023/04/11/image-24.png){: w="300" }

```
General Settings:
Client type: OpenID Connector
Client ID: oauth2proxy # Понадобится при настройке OAuth2 Proxy
```

![](/assets/img/posts/2023/04/11/image-25.png){: w="300" }

```
Capability config:
Client authentication: On
Authentication flow: Standard flow, Direct access grants, OAuth 2.0 Device Authorization Grant, OIDC CIBA Grant
```

![](/assets/img/posts/2023/04/11/image-26.png){: w="300" }

```
Login settins:
Root URL: http://prometheus.itdraft.ru/
Home URL: http://prometheus.itdraft.ru/
Valid redirect URIs: http://prometheus.itdraft.ru/oauth2/callback
```

Сохраняемся, переходим в настройки созданного клиента

Копируем `Client secret`, он нам понадобится при настройке OAuth2 Proxy

![](/assets/img/posts/2023/04/11/image-28.png){: w="300" }

Создаем Mapper

![](/assets/img/posts/2023/04/11/image-27.png){: w="300" }

```
Client scopes > oauth2proxy-dedicated
```

![](/assets/img/posts/2023/04/11/image-29.png){: w="300" }

```
Add mapper > By Configuration > Audience
```

![](/assets/img/posts/2023/04/11/image-30.png){: w="300" }

```sh
Mapper type: Audience
Name: static audience
Included Client Audience: oauth2proxy
Included Custom Audience: oauth2proxy
Add to access token: On
```

Настройка клиента Keycloak завершена

## Установка OAuth2 Proxy и настройка в режиме Reverse Proxy

Создаем пользователя и необходимые каталоги
```sh
$ sudo useradd -M -s /bin/false oauth2proxy
$ sudo mkdir -p /opt/oauth2-proxy
```

Скачиваем финальную верcию OAuth2 Proxy, распаковываем её и переходим в каталог
```sh
$ wget https://github.com/oauth2-proxy/oauth2-proxy/releases/download/v7.4.0/oauth2-proxy-v7.4.0.linux-amd64.tar.gz
$ tar xfz oauth2-proxy-v7.4.0.linux-amd64.tar.gz
$ cd oauth2-proxy-v7.4.0.linux-amd64
```

Перемещаем исполняемый файл, меняем владельца файла
```sh
$ sudo cp oauth2-proxy /usr/local/bin/
$ sudo chown oauth2proxy. /usr/local/bin/oauth2-proxy
```

Генерим ключ, который затем пропишем в конфигурационном файле
```sh
$ openssl rand -base64 32 | tr -- '+/' '-_'
YL8TS48lmrepWgfLERXmFp1TXXzUyXrB9FxoCqnBPfk=
```

Создаем конфигурационный файл `oauth2-proxy.cfg`

```sh
$ sudo nano /opt/oauth2-proxy/oauth2-proxy.cfg
provider="keycloak-oidc"
provider_display_name="Keycloak"
client_id="oauth2proxy" # из настроек keycloak
client_secret="AJZGOuYcLoFxrd5sgFDsScmrrhyS69s0" # из настроек keycloak
redirect_url="http://prometheus.itdraft.ru/oauth2/callback"
oidc_issuer_url="https://keycloak.itdraft.ru:8443/realms/myrealm"
cookie_secure="false"
cookie_secret="YL8TS48lmrepWgfLERXmFp1TXXzUyXrB9FxoCqnBPfk="
cookie_domains=[".itdraft.ru"]
#upstreams=["http://127.0.0.1:9090/"] # My website server
email_domains=["*"]
http_address="127.0.0.1:4180"
reverse_proxy="true"
ssl_insecure_skip_verify="true" # пропускаем проверку ssl
cookie_expire="30m"
session_cookie_minimal="true"
scope="openid"
whitelist_domains=[".itdraft.ru"]
insecure_oidc_allow_unverified_email="true" # Пропустить проверку валидности email
```

Меняем владельца файла
```sh
$ sudo chown oauth2proxy. /opt/oauth2-proxy/oauth2-proxy.cfg
```

Создаем системный юнит для OAuth2 Proxy
```sh
$ sudo nano /etc/systemd/system/oauth2-proxy.service
[Unit]
Description=oauth2-proxy daemon service
After=syslog.target network.target

[Service]
# www-data group and user need to be created before using these lines
User=oauth2proxy
Group=oauth2proxy

ExecStart=/usr/local/bin/oauth2-proxy --config=/opt/oauth2-proxy/oauth2-proxy.cfg
ExecReload=/bin/kill -HUP $MAINPID

KillMode=process
Restart=always

[Install]
WantedBy=multi-user.target
```

Перечитываем юниты, добавляем сервис а автозагрузку и запускаем его
```sh
$ sudo systemctl daemon-reload
$ sudo systemctl enable --now oauth2-proxy
$ sudo systemctl status oauth2-proxy
```

## Установка и настройка Nginx

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

Устанавливаем Nginx и отключаем дефолтный конфиг
```sh
$ sudo dnf -y install nginx
$ cd /etc/nginx/conf.d/
$ sudo mv default.conf default.conf.disable
```

Создаем конфиг `prometheus.conf`
```sh
$ sudo nano /etc/nginx/conf.d/prometheus.conf
server {
  listen 80;
  server_name _;

  location /oauth2/ {
    proxy_pass       http://127.0.0.1:4180;
    proxy_set_header Host                    $host;
    proxy_set_header X-Real-IP               $remote_addr;
    proxy_set_header X-Scheme                $scheme;
    proxy_set_header X-Auth-Request-Redirect $request_uri;
  }
  location = /oauth2/auth {
    proxy_pass       http://127.0.0.1:4180;
    proxy_set_header Host             $host;
    proxy_set_header X-Real-IP        $remote_addr;
    proxy_set_header X-Scheme         $scheme;
    proxy_set_header Content-Length   "";
    proxy_pass_request_body           off;
  }

  location ^~ /alertmanager/ {
    auth_request /oauth2/auth;
    error_page 401 = /oauth2/sign_in;

    auth_request_set $user   $upstream_http_x_auth_request_user;
    auth_request_set $email  $upstream_http_x_auth_request_email;
    proxy_set_header X-User  $user;
    proxy_set_header X-Email $email;

    auth_request_set $auth_cookie $upstream_http_set_cookie;
    add_header Set-Cookie $auth_cookie;

    proxy_pass http://127.0.0.1:9093/;
  }

  location ^~ /prometheus/ {
    auth_request /oauth2/auth;
    error_page 401 = /oauth2/sign_in;

    auth_request_set $user   $upstream_http_x_auth_request_user;
    auth_request_set $email  $upstream_http_x_auth_request_email;
    proxy_set_header X-User  $user;
    proxy_set_header X-Email $email;

    auth_request_set $auth_cookie $upstream_http_set_cookie;
    add_header Set-Cookie $auth_cookie;

    proxy_pass http://127.0.0.1:9090/;
  }
}
```

Добавляем сервис Nginx в автозагрузку, запускаем его
```sh
$ sudo systemctl enable --now nginx
$ sudo systemctl status nginx
```

Открываем порт 80/tcp
```sh
$ sudo firewall-cmd --permanent --zone=public --add-port=80/tcp
$ sudo firewall-cmd --reload
```

## Настройка Prometheus в режиме Reverse Proxy

Редактируем системный юнит
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
    --config.file /opt/prometheus/prometheus.yml \
    --storage.tsdb.path /var/lib/prometheus/ \
    --web.console.templates=/opt/prometheus/consoles \
    --web.console.libraries=/opt/prometheus/console_libraries \
    --web.listen-address="127.0.0.1:9090" \
    --web.external-url="http://prometheus.itdraft.ru/prometheus/" \
    --web.route-prefix="/"

[Install]
WantedBy=multi-user.target
```

Перечитываем юниты, перезапускаем сервис
```sh
$ sudo systemctl daemon-reload
$ sudo systemctl restart prometheus
```

Закрываем в файрволле порт 9090/tcp
```sh
$ sudo firewall-cmd --permanent --zone=public --remove-port=9090/tcp
$ sudo firewall-cmd --reload
```

## Настройка Alertmanager в режиме Reverse Proxy

Редактируем системный юнит
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
    --config.file=/opt/alertmanager/alertmanager.yml \
    --storage.path=/var/lib/alertmanager \
    --web.listen-address="127.0.0.1:9093" \
    --web.external-url="http://prometheus.itdraft.ru/alertmanager/" \
    --web.route-prefix="/"
    $ALERTMANAGER_OPTS
ExecReload=/bin/kill -HUP $MAINPID
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

Перечитываем юниты, перезапускаем сервис
```sh
$ sudo systemctl daemon-reload
$ sudo systemctl restart alertmanager
```

Закрываем в файрволле порт `9093/tcp`
```sh
$ sudo firewall-cmd --permanent --zone=public --remove-port=9093/tcp
$ sudo firewall-cmd --reload
```

Настройка завершена. Переходим на наш сайт, авторизуемся, получаем доступ

![](/assets/img/posts/2023/04/11/image-31.png){: w="300" }

![](/assets/img/posts/2023/04/11/image-32.png){: w="300" }

![](/assets/img/posts/2023/04/11/image-33.png){: w="300" }

![](/assets/img/posts/2023/04/11/image-34.png){: w="300" }
