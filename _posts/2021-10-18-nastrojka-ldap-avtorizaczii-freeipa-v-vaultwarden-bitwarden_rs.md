---
title: "Настройка LDAP-авторизации (FreeIPA) в Vaultwarden (Bitwarden_RS)"
date: "2021-10-18"
categories: 
  - Linux
  - Vaultwarden
tags: 
  - "bitwarden"
  - "bitwarden_rs"
  - "centos"
  - "freeipa"
  - "ldap"
  - "rocky-linux"
  - "rust"
  - "vaultwarden"
image:
  path: /commons/laptop-2.jpg
  alt: "LDAP-авторизация в Vaultwarden"
---

> **Vaultwarden (Bitwarden_RS)** - менеджер паролей с открытым кодом. Легковесный форк широко известного Bitwarden, написан на Rust. В качестве БД используется SQLite, MariaDB, PostgreSQL

В предыдущей статье была рассмотрена установка менеджера паролей [Vaultwarden (Bitwarden_RS)]({% post_url 2021-10-14-ustanovka-vaultwarden-bitwarden_rs-i-postgresql-ne-v-docker-ispolnenii-v-centos-8-rocky-linux %})

Переключимся на пользователя Vaultwarden

```sh
$ sudo su - vaultwarden
```

Скачиваем модуль ldap-авторизации для Vaultwarden из репозитория и устанавливаем его

```sh
$ wget https://github.com/ViViDboarder/vaultwarden_ldap/archive/refs/tags/v0.4.0.tar.gz
$ tar xzvf v0.4.0.tar.gz
$ cd vaultwarden_ldap-0.4.0/
$ cargo new --bin vaultwarden_ldap
$ cargo build --locked --release
```

Копируем конфиг

```sh
$ cp /opt/vaultwarden/vaultwarden_ldap-0.4.0/example.config.toml /opt/vaultwarden/vaultwarden_ldap-0.4.0/target/release/config.toml
```

Переключаемся на sudo-пользователя

```sh
$ exit
```

Правим конфиг для `vaultwarden_ldap`

```sh
$ sudo nano /opt/vaultwarden/vaultwarden_ldap-0.4.0/target/release/config.toml
vaultwarden_url = "https://vault.itdraft.ru"
vaultwarden_admin_token = "Q8rKXS3l6jmUYrcJGlwueZhiiIZWteGMVZe7Db/qFe0nQ68C5P5H4Bdi/1AMv4xU"
ldap_host = "192.168.11.201"
ldap_bind_dn = "uid=myuser,cn=users,cn=accounts,dc=example,dc=local"
ldap_bind_password = "mupass"
ldap_search_base_dn = "cn=users,cn=accounts,dc=example,dc=local"
ldap_search_filter = "(&(objectClass=posixAccount)(memberOf=cn=vaultwarden,cn=groups,cn=accounts,dc=example,dc=local))"
ldap_sync_interval_seconds = 10
```

В данном конфиге указано, что пользователям из группы vaultwarden будут иметь доступ в Vaultwarden (им будет отправлено письмо при запуске сервиса)

Создадим Systemd Unit

```sh
$ sudo nano /etc/systemd/system/vaultwarden-ldap.service

[Unit]
Description=Bitwarden LDAP (Rust Edition)
Documentation=https://github.com/ViViDboarder/vaultwarden_ldap

After=network.target mariadb.service
Requires=vaultwarden.service

[Service]
# The user/group bitwarden_rs is run under. the working directory (see below) should allow write and read access to this user/group
User=vaultwarden
Group=vaultwarden
# The location of the .env file for configuration
EnvironmentFile=/opt/vaultwarden/vaultwarden_ldap-0.4.0/target/release/config.toml
# The location of the compiled binary
ExecStart=/opt/vaultwarden/vaultwarden_ldap-0.4.0/target/release/vaultwarden_ldap
# Only allow writes to the following directory and set it to the working directory (user and password data are stored here)
WorkingDirectory=/opt/vaultwarden/vaultwarden_ldap-0.4.0/target/release/
ReadWriteDirectories=/opt/vaultwarden/vaultwarden_ldap-0.4.0/target/release/

[Install]
WantedBy=multi-user.target
```

Запускаем сервис, проверяем статус

```sh
$ sudo systemctl daemon-reload
$ sudo systemctl enable vaultwarden-ldap --now

$ sudo systemctl status vaultwarden-ldap
```