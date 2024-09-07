---
title: "Nginx Reverse Proxy для HashiCorp Vault в Centos 8"
date: "2020-12-03"
categories: 
  - Web
  - Security-System
tags: 
  - "hashicorp-vault"
  - "nginx"
  - "reverse-proxy"
image:
  path: /commons/752128_6d9f_2.jpg
  alt: "Nginx Reverse Proxy для HashiCorp Vault"
---

> Обратный прокси-сервер (**reverse proxy**) — тип прокси-сервера, который ретранслирует запросы клиентов из внешней сети на один или несколько серверов, логически расположенных во внутренней сети. При этом для клиента это выглядит так, будто запрашиваемые ресурсы находятся непосредственно на прокси-сервере.
{: .prompt-tip }

- Список статей из категории [HashiCorp Vault](/tags/hashicorp-vault/)

В предыдущей статье мы рассматривали установку HashiCorp Vault и настройку собственного центра сертификации Vault PKI.
После того, как в собственном центре сертификации мы выпустили ssl-сертификат для домена `vault.example.com`, перенастроим Vault на использование ssl

## Установка Nginx

Установим утилиту dnf-utils

```sh
$ sudo dnf -y install dnf-utils
```

Добавим репозиторий Nginx

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

Устанавливаем Nginx

```sh
$ sudo dnf -y install nginx
```

Запускам сервис, добавляем его в автозагрузку и проверяем статус

```sh
$ sudo systemctl enable --now nginx
$ systemctl status nginx
```

## Настраиваем Firewall

Открываем порты 80/443

```sh
$ sudo firewall-cmd --zone=public --add-port=80/tcp --permanent
$ sudo firewall-cmd --zone=public --add-port=443/tcp --permanent
$ sudo firewall-cmd --reload
```

## Настройка Nginx

Отключаем дефолтный конфиг

```sh
$ sudo mv /etc/nginx/conf.d/default.conf /etc/nginx/conf.d/default.conf_disabled
```

Создаем новый конфиг Nginx для Vault

```sh
$ sudo nano /etc/nginx/conf.d/vault.example.com.conf
server {
  listen 80 default_server;
  server_name _;
  return 301 https://$host$request_uri;
}

server {
  listen 443 ssl default_server;
  server_name _;

  ssl on;
  ssl_certificate /etc/nginx/conf.d/vault.example.com.crt.pem;
  ssl_certificate_key /etc/nginx/conf.d/vault.example.com.crt.key;

  ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
  ssl_ciphers HIGH:!aNULL:!MD5;

  ssl_prefer_server_ciphers on;
  ssl_session_timeout 1d;
  ssl_session_cache shared:SSL:50m;
  ssl_session_tickets off;

  location / {
    proxy_pass http://127.0.0.1:8200;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto https;
  }
}
```

Копируем сертификаты, которые создали раньше

```sh
$ sudo cp ~/vault.example.com.crt.* /etc/nginx/conf.d/
```

> При создании PKI мы указывали URL корневого и промежуточного центра сертификации без https и с портом (http://vault.example.com:8200). Через веб-интерфейс (или через терминал) надо поменять URL для корневого центра сертификации (https://vault.example.com), удалить промежуточный центр сертификации и пересоздать его с правильными URL(с момента где мы создаем CSR-файл и до конца).
> 
> Таким образом корневой центр сертификации (который мы распространяем) у нас остается прежний, а промежуточный и сертификат сервера - новые.
> 
> Либо на этапе создания собственного центра сертификации можно учесть этот момент, что будет правильнее.
{: .prompt-info }

Перезапускаем Nginx

```sh
$ sudo systemctl restart nginx
```

## Настройка Vault

Редактируем конфиг Vault

```sh
$ sudo nano /etc/vault.d/vault.hcl
disable_cache = true
disable_mlock = true
ui = true
listener "tcp" {
    address          = "127.0.0.1:8200"
    tls_disable      = 1
}
# listener "tcp" {
#     tls_disable = 0
#     address     = "0.0.0.0:8200"
#     tls_cert_file = "/etc/vault.d/vault.example.com.crt.pem"
#     tls_key_file = "/etc/vault.d/vault.example.com.crt.key"
# }
# storage "file" {
#     path  = "/var/lib/vault/data"
# }
storage "postgresql" {
    connection_url = "postgres://vltusr:dnTkb35qNURNeV@localhost:5432/vaultdb?sslmode=disable"
    table          = "vault_kv_store"
    max_parallel   = "128"
}
api_addr         = "http://127.0.0.1:8200"
max_lease_ttl         = "10h"
default_lease_ttl    = "10h"
cluster_name         = "vault"
raw_storage_endpoint     = true
disable_sealwrap     = true
disable_printable_check = true
```

Перезапускаем Vault

```sh
$ sudo systemctl restart vault
```

Экспортируем переменную

```sh
$ export VAULT_ADDR=http://127.0.0.1:8200
```

Редактируем файл .bashrc

```sh
$ nano ~/.bashrc
[…]
export VAULT_ADDR=http://127.0.0.1:8200
```

Так же можно удалить строку с нашим хостом и ip-адресом из файла /etc/hosts

Проверяем статус, распаковываем хранилище, авторизуемся

```sh
$ vault status
$ vault operator unseal
$ vault login
```

## Продолжение настройки Firewall

Закрываем порт 8200

```sh
$ sudo firewall-cmd --zone=public --remove-port=8200/tcp --permanent
$ sudo firewall-cmd --reload
```
