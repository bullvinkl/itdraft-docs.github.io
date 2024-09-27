---
title: "Храним секреты Ansible в Hashicorp Vault"
date: "2021-03-26"
categories: 
  - DevOps
  - Security-System
tags: 
  - "ansible"
  - "hashicorp-vault"
image:
  path: /commons/laptop-2.jpg
  alt: "Секреты Ansible в Hashicorp Vault"
---

> **Hashicorp Vault** - open-source инструмент для управления секретами. Vault написан на Go и распространяется как бинарник.
> При использовании в связки с Ansible, секреты хранятся в Vault в key-value виде.
{: .prompt-tip }

## Подготовка Vault

Распечатываем хранилище

```sh
$ vault operator unseal
```

Авторизуемся

```sh
$ vault login
Token (will be hidden):
```

Включаем хранилище секретов key-volume версии 1

```sh
$ vault secrets enable -tls-skip-verify -path=ansible kv
```

Записываем данные в хранилище

```sh
$ vault kv put -tls-skip-verify ansible/serverlab/production/db \
   dbusername=wp_user \
   dbpassword=passwd123 \
   dbhost=vault.exampe.local \
   dbname=blog
```

Выполним следующую команду, чтобы проверить

```sh
$ vault kv get ansible/serverlab/production/db
======= Data =======
Key           Value
--- -----
dbhost        vault.exampe.local
dbname        blog
dbpassword    passwd123
dbusername    wp_user
```

Делаем политику "только чтение"

Создадим файл ansible.hcl

```sh
$ nano ansible.hcl
path "ansible/*" {
  capabilities = [ "read", "list" ]
}
```

Записываем политику "ansible"

```sh
$ vault policy write ansible ansible.hcl
Success! Uploaded policy: ansible
```

Создадим токен, чтобы мы могли пройти аутентификацию

```sh
$ vault token create -policy="ansible"
Key                  Value
--- -----
token                s.7JjtwBWjKrtNsP2pYbwNv9er
token_accessor       JpMeUksuVRuRq0lSW07KRmLi
token_duration       10h
token_renewable      true
token_policies       ["ansible" "default"]
identity_policies    []
policies             ["ansible" "default"]
```

## Подготовка Ansible

Устанавливаем `ansible` и плагин `hashi_vault`

```sh
$ sudo dnf install ansible
$ sudo pip3 install hvac
```

Создаем каталог для локального плэйбука и переходим в него

```sh
$ mkdir ansible-test && cd ansible-test
```

Создаем файл с нашим локальным хостом

```sh
$ nano hosts
[servers]
vault.example.local ansible_ssh_host=127.0.0.1
```

Создаем плэйбук

```sh
$ nano playbook.yml
```
```yaml
---
- hosts: all
  connection: local
  vars:
    vault_url: https://192.168.11.200
    vault_token: s.7JjtwBWjKrtNsP2pYbwNv9er
    vault_secret: ansible/serverlab/production/db
    serverlab_db: " {{ lookup('hashi_vault', 'secret= {{ vault_secret }} token= {{ vault_token }} url= {{ vault_url }}') }} "
  roles:
    - wordpress
```

```sh
$ mkdir -p roles/wordpress/{tasks,templates}
```

```sh
$ nano roles/wordpress/tasks/main.yml
```
```yaml
---
- name: Configure WordPress
  template:
    src: wp-config.php.j2
    dest: /tmp/wp-config.php
  with_items:
    - "{{ serverlab_db }}"
  tags:
    - update_wp_config

- name: Return all secrets from a path
  debug:
    msg: " {{ lookup('hashi_vault', 'secret= {{ vault_secret }} token= {{ vault_token }} url= {{ vault_url }} validate_certs=False') }} "
  tags:
    - test
```

```sh
$ nano roles/wordpress/templates/wp-config.php.j2
define('DB_NAME', ' {{ serverlab_db.dbname }} ');
define('DB_USER', ' {{ serverlab_db.dbusername }} ');
define('DB_PASSWORD', ' {{ serverlab_db.dbpassword }} ');
define('DB_HOST', ' {{ serverlab_db.dbhost }} ');
```

Запускаем плэйбук

```sh
$ ansible-playbook -i hosts -l servers playbook.yml --tags "test"
$ ansible-playbook -i hosts -l servers playbook.yml --tags "update_wp_config"
```

Первая команда выведет результат в консоль. Вторая команда вытянет данные из Vault и сохранит их в файл

Проверяем

```sh
$ cat /tmp/wp-config.php
define('DB_NAME', 'blog');
define('DB_USER', 'wp_user');
define('DB_PASSWORD', 'passwd123');
define('DB_HOST', 'vault.exampe.local');
```
