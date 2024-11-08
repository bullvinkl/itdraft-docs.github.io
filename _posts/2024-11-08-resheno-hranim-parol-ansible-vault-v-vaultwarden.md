---
layout: post
title: "[Решено] Храним пароль Ansible Vault в Vaultwarden"
date: "2024-11-08"
categories:
  - Security-System
  - DevOps
tags:
  - vaultwarden
  - bitwarden
  - ansible
  - ansible-vault
image:
  path: /commons/password-strength-in-2019-two-factor-authentication.png
  alt: "Храним пароль Ansible Vault в Vaultwarden"
---

> **Ansible Vault** - механизм шифрования переменных и файлов, который помогает спрятать секретные данные, такие как пароли или ключи к разнообразным сервисам.
> **Vaultwarden** — это менеджер паролей с открытым исходным кодом, который позволяет безопасно хранить и управлять вашими паролями, логинами и другими конфиденциальными данными.
{: .prompt-tip }

- [Установка Vaultwarden и PostgreSQL в Debian]({% post_url 2024-11-06-ustanovka-vaultwarden-i-postgresql-v-debian-no-docker %}){:target="_blank"} была рассмотрена в одной из предыдущих статей.
- [Использование Ansible-vault]({% post_url 2024-10-08-ispolzovaniye-ansible-vault-v-ansible-playbook-i-sokhraneniye-v-git %}){:target="_blank"} так же было рассмотрена в одной из предыдущих статей.
- Список статей из категории [Vaultwarden](/tags/vaultwarden/){:target="_blank"}.

## Подготовка

Устанавливаем Ansible
```bash
$ sudo apt install ansible
```

Устанавливаем коллекцию `community.general` для Ansible
```bash
$ ansible-galaxy collection install community.general
```

Скачиваем консольный клиент `bitwarden` и распаковываем его
```bash
$ wget https://github.com/bitwarden/clients/releases/download/cli-v2024.10.0/bw-linux-2024.10.0.zip
$ unzip bw-linux-2024.10.0.zip
```

Устанавливаем консольный клиент `bitwarden`
```bash
$ sudo install bw /usr/local/bin/
$ which bw
```

Задаем адрес нашел сервера Vaultwarden

```bash
$ bw config server https://vault.itdraft.ru
```

Пробуем авторизоваться 

```bash
$ bw login m.makarov@itdraft.ru
```

После авторизации, консоль выдаст нам переменную среды. Выполним команду, что бы постоянно не использовать логин,пароль
```bash
$ export BW_SESSION="6tF1rCk6w0yDiMhpDFHkoLvW/kQHJGr0Y1d3BdIGrcvO4pGzZuIbnjauDXaAhoIqrUUrnI+MxxNZNlLMwXCFOA=="
```
Если мы забыли запустить (или потерять) эти команды, можно получить их с помощью команды `bw unlock`

Cинхронизируем хранилище
```bash
$ bw sync
```

Не забываем время от времени запускать `bw sync`, чтобы подтянулась свежая копия хранилища.

## Реализация

Сделаем скрипт `ansible-vault-pass.sh`, который выполняет разблокировку хранилища Vaultwarden, а затем ищет и возвращает пароль Ansible Vault
```bash
sudo nano ansible-vault-pass.sh
```

Содержимое скрипта
```bash
#!/bin/bash

_BW_VAULT_ID="ansible-vault"
_bw_session="$(bw unlock --raw)"
echo "$(bw get password ${_BW_VAULT_ID} --session ${_bw_session} --raw)"
```

- запись `ansible-vault` должна присутствовать в хранилище Vaultwarden

Теперь при запуске `ansible-playbook` с параметром `--vault-password-file` будет запрашивать пароль Ansible Vault из Vaultwarden
```bash
ansible-playbook myplaybook.yml --vault-password-file=~./ansible-vault-pass.sh
```

Параметру `--vault-password-file` можно задать значение по умолчанию в основной конфигурации Ansible (обычно `~/.ansible.cfg`)
```bash
sudo nano ~/.ansible.cfg
```

Содержимое конфигурации
```ini
[defaults]
vault_password_file=/path/to/ansible-vault-pass.sh
```
