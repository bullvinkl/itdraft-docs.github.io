---
title: "Установка HashiCorp Vault в Centos 8"
date: "2020-11-27"
categories: 
  - Linux
  - HashiCorp-Vault
tags: 
  - "centos"
  - "hashicorp-vault"
  - "selinux"
image:
  path: /commons/cloud-provider.png
  alt: "Установка HashiCorp Vault"
---

> **HashiСorp Vault** — это инструмент с открытым исходным кодом, предназначенный для безопасного хранения секретов и конфиденциальных данных в динамических облачных средах. Он обеспечивает надежное шифрование данных, доступ на основе идентификации с помощью настраиваемых политик

Рассмотрим вариант установки программного обеспечения HashiCorp Vault с файловым типом хранения данных (секретов).

## Подготовка

Добавляем hostname и ip в файл /etc/hosts

```sh
$ sudo nano /etc/hosts
[…]
192.168.11.200    vault.example.com
```

Обновляем ОС, ставим необходимый софт

```sh
$ sudo dnf -y update
$ sudo dnf -y install unzip nano
```

## Установка HashiСorp Vault

Переходим в каталог /tmp

```sh
$ cd /tmp
```

Скачиваем финальную версию HashiСorp Vault, распаковываем ее

```sh
$ curl -L https://releases.hashicorp.com/vault/1.6.0/vault_1.6.0_linux_amd64.zip -o /tmp/vault_1.6.0_linux_amd64.zip
$ unzip vault_1.6.0_linux_amd64.zip
```

Меняем владельца и переносим файл

```sh
$ sudo chown root:root vault
$ sudo mv vault /usr/local/bin/
```

Проверяем версию

```sh
$ vault --version
Vault v1.6.0 (7ce0bd9691998e0443bc77e98b1e2a4ab1e965d4)
```

Включаем авто заполнение команд

```sh
$ vault -autocomplete-install
$ complete -C /usr/local/bin/vault vault
```

## Настройка HashiСorp Vault

Создаем системные каталоги для HashiСorp Vault

```sh
$ sudo mkdir -p /etc/vault.d /var/lib/vault/data
```

`/var/lib/vault/data` - если тип хранилища файловый.

Позже рассмотрим подключение типа хранилища на СУБД PostgreSQL

Создаем системного пользователя HashiСorp Vault

```sh
$ sudo useradd --system --home /etc/vault.d --shell /bin/false vault
$ sudo chown -R vault:vault /etc/vault.d /var/lib/vault/
$ sudo chmod 755 /etc/vault.d /var/lib/vault
```

Создаем Systemd Unit

```sh
$ sudo nano /etc/systemd/system/vault.service
[Unit]
Description="HashiCorp Vault - A tool for managing secrets"
Documentation=https://www.vaultproject.io/docs/
Requires=network-online.target
After=network-online.target
ConditionFileNotEmpty=/etc/vault.d/vault.hcl
StartLimitIntervalSec=60
StartLimitBurst=3

[Service]
User=vault
Group=vault
ProtectSystem=full
ProtectHome=read-only
PrivateTmp=yes
PrivateDevices=yes
SecureBits=keep-caps
AmbientCapabilities=CAP_IPC_LOCK
Capabilities=CAP_IPC_LOCK+ep
CapabilityBoundingSet=CAP_SYSLOG CAP_IPC_LOCK
NoNewPrivileges=yes
ExecStart=/usr/local/bin/vault server -config=/etc/vault.d/vault.hcl
ExecReload=/bin/kill --signal HUP $MAINPID
KillMode=process
KillSignal=SIGINT
Restart=on-failure
RestartSec=5
TimeoutStopSec=30
StartLimitInterval=60
StartLimitIntervalSec=60
StartLimitBurst=3
LimitNOFILE=65536
LimitMEMLOCK=infinity

[Install]
WantedBy=multi-user.target
```

Создаем конфигурационный файл для HashiСorp Vault

```sh
$ sudo nano /etc/vault.d/vault.hcl
disable_cache = true
disable_mlock = true
ui = true

listener "tcp" {
    tls_disable      = 1
#   address          = "0.0.0.0:8200"
    address          = "vault.example.com:8200"
}

# отключаем ssl
# listener "tcp" {
#    tls_disable = 0
#    address     = "vault.example.com:8200"
#    tls_cert_file = "/etc/ssl/cert/vault.example.com.crt"
#    tls_key_file = "/etc/ssl/private/vault.example.com.key"
# }

storage "file" {
   path  = "/var/lib/vault/data"
}

# api_addr         = "http://0.0.0.0:8200"
api_addr         = "http://vault.example.com:8200"
max_lease_ttl         = "10h"
default_lease_ttl    = "10h"
cluster_name         = "vault"
raw_storage_endpoint     = true
disable_sealwrap     = true
disable_printable_check = true
```

## Настройка SeLinux

Добавляем порт Vault в исключения

```sh
$ cd /tmp
$ sudo grep vault /var/log/audit/audit.log | grep denied | audit2allow -m vaultlocalconf > vaultlocalconf.te
$ sudo grep vault /var/log/audit/audit.log | grep denied | audit2allow -M vaultlocalconf

    ******************** IMPORTANT ***********************
    To make this policy package active,

    execute: semodule -i vaultlocalconf.pp

$ sudo semodule -i vaultlocalconf.pp
```

Добавляем сервис в автозагрузку и запускаем его

```sh
$ sudo systemctl daemon-reload
$ sudo systemctl enable --now vault
```

Проверяем статус

```sh
$ systemctl status vault
```

## Настройка Firewall

Открываем порт 8200/tcp

```sh
$ sudo firewall-cmd --zone=public --add-port=8200/tcp --permanent
$ sudo firewall-cmd --reload
```

## Инициализация HashiCorp Vault Server

Добавляем переменные, что б в дальнейшем не вводить ее каждый раз в -address=http://vault.example.com:8200

```sh
$ export PATH=$PATH:/usr/local/bin
$ echo "export PATH=$PATH:/usr/local/bin" >> ~/.bashrc
$ export VAULT_ADDR=http://vault.example.com:8200
$ echo "export VAULT_ADDR=http://vault.example.com:8200" >> ~/.bashrc
```

Инициализируем сервис с сохранением ключей в файл /etc/vault.d/init.file (не безопасно)

```sh
$ vault operator init -n 5 -t 3 | sudo tee /etc/vault.d/init.file
Unseal Key 1: +DTPnUxqmspBKng0wVHeHc59pXnZJGKxMsSuX+CrQ91E
Unseal Key 2: h8ogFxApCpwRqfK4Fz0frtFM64t2ldFAZLJhvapJBqnl
Unseal Key 3: MTbJX9HVN0A1dOd7hH+L1wcEus9M2C0gK9nrqsoXCtYz
Unseal Key 4: H4B2mlx+1uRg70uySY/KGQ86hsjlrCiSVD4gyoO6tj5V
Unseal Key 5: 3QKfJ/c4v65eEej5K9OvQseo3sETRhN2DZVRXkg5d8Wh
Initial Root Token: s.2xxinhG13tTgpHrxb8EyAgL5
...
```

где  
`-n (-key-share)` - количество общих ключей, на которые нужно разделить сгенерированный главный ключ. Это количество «ключей распечатки», которое нужно сгенерировать.  
`-t (-key-threshold)` - количество общих ключей, необходимых для восстановления главного ключа. Это должно быть меньше или равно `-key-share`

Проверяем статус

```sh
$ vault status
Key                Value
--- -----
Seal Type          shamir
Initialized        true
Sealed             true <--- vault запечатан, надо расечатывать (вводить 3 разных Unseal Key)
Total Shares       5 <--- всего ключей
Threshold          3 <--- ключей, для распечатывания
Unseal Progress    0/3
Unseal Nonce       n/a
Version            1.6.0
Storage Type       file <--- тип хранилища файловое
HA Enabled         false
```

Распечатываем хранилище, иначе не получится авторизоваться

```sh
$ vault operator unseal
Unseal Key (will be hidden):
$ vault operator unseal
Unseal Key (will be hidden):
$ vault operator unseal
Unseal Key (will be hidden):
```

Смотрим статус

```sh
$ vault status
Key             Value
--- -----
Seal Type       shamir
Initialized     true
Sealed          false <--- Распечатано
Total Shares    5
Threshold       3
Version         1.6.0
Storage Type    file
Cluster Name    vault
Cluster ID      d5ec1a04-2988-d1fa-eb16-2bbf6f47d3f2
HA Enabled      false
```

Авторизуемся

```sh
$ vault login
Token (will be hidden): s.2xxinhG13tTgpHrxb8EyAgL5
```

Распечатывать хранилище и авторизоваться можно так же через [web-интерфейс](http://vault.example.com:8200)

Если надо удалить базу (файловую)

```sh
$ sudo systemctl stop vault
$ sudo rm -rf  /var/lib/vault/data/*
$ sudo systemctl start vault
```