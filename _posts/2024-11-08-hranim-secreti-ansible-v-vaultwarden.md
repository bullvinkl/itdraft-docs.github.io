---
layout: post
title: "Храним секреты для Ansible в Vaultwarden"
date: "2024-11-08"
categories:
  - Security-System
  - DevOps
tags:
  - vaultwarden
  - bitwarden
  - ansible
image:
  path: /commons/hack_b-min-1.png
  alt: "Храним секреты для Ansible в Vaultwarden"
---

> **Ansible** — система управления конфигурациями, написанная на языке программирования Python, с использованием декларативного языка разметки для описания конфигураций. Применяется для автоматизации настройки и развёртывания программного обеспечения. Обычно используется для управления Linux-узлами, но Windows также поддерживается.
> **Vaultwarden** — это менеджер паролей с открытым исходным кодом, который позволяет безопасно хранить и управлять вашими паролями, логинами и другими конфиденциальными данными.
{: .prompt-tip }

- Установка консольного клиента [bitwarden]({% post_url 2024-11-08-resheno-hranim-parol-ansible-vault-v-vaultwarden %}){:target="_blank"} была рассмотрена в одной из прошлых статей. 
- Список статей из категории [Vaultwarden](/tags/vaultwarden/){:target="_blank"}.

## Полезные команды для работы с клиентом bitwarden

| Команда | Описание |
|---|---|
| `bw login EMAIL PASSWORD` | авторизация по логину и паролю |
| `bw unlock` | разблокировака хранилища |
| `bw unlock --passwordfile ./mp.txt` | разблокировака без ввода master password |
| `bw sync` | синхронизация хранилища |
| `bw list items` | показать все записи |
| `bw list collections` | показать все коллекции |
| `bw list items --search 'name'` | поиск записи `name` |
| `bw get password test_name` | показать пароль записи `test_name` |
| `bw lock` | заблокировать хранилище |
| `bw config server https://vault.itdraft.ru` | указать свой сервер |

## Пример 1. Вытащить данные

Создадим `ansible-playbook` `playbook.yml` с содержимым
```yaml
---
- hosts: localhost
  gather_facts: false
  connection: local
  tasks:
    - name: "Get password from Vaultwarden"
      debug:
        msg: >-
          - login: { { lookup('community.general.bitwarden', 'test_name', field='username') } }
          - password: { { lookup('community.general.bitwarden', 'test_name', field='password') } }
```

> Фигурные скобки `{ {` и `} }` следует писать без пробела между ними. Движок сайта некорректно их обрабатывает.
{: .prompt-info }

Выполним его
```bash
$ ansible-playbook playbook.yml
PLAY [localhost] *************************************************************************************************************************************

TASK [Get password from Vaultwarden] *************************************************************************************************************
ok: [localhost] => {
    "msg": "- login: ['test_user'] - passwd: ['test_password']"
}

PLAY RECAP *******************************************************************************************************************************************
localhost                  : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

Как видно из примера, у нас получилось достать параметрый записи `test_name` из хранилища Vaultwarden

## Пример 2. ansible_become_pass

Создадим `ansible-playbook` `playbook2.yml` с содержимым
```yaml
---
- hosts: all
  become: true
  tasks:
    - name: "Create file"
      file:
        path: "/home/user/ansible/test.txt"
        state: touch
```

Создадим файл инвентаризации
```bash
$ nano hosts
[tst]
test-srv ansible_ssh_host=127.0.0.1 ansible_user=user ansible_ssh_pass=user ansible_become_pass="{ { lookup('community.general.bitwarden', 'test_name', field='password')[0] } }"
```

> Фигурные скобки `{ {` и `} }` следует писать без пробела между ними. Движок сайта некорректно их обрабатывает.
{: .prompt-info }

Я указал `ansible_user` и `ansible_ssh_pass` в явном виде, т.к. это демо-стенд без настроенной авторизации по сертификату.

> В параметре `ansible_become_pass` указана строка запроса `{ { lookup('community.general.bitwarden', 'test_name', field='password')[0] } }`, оканчивающаяся на `[0]`, т.к. без этого возвращаемый объект запроса без `[0]` представляет собой массив. А нам нужен не массив, а значение!
{: .prompt-warning }

> Фигурные скобки `{ {` и `} }` следует писать без пробела между ними. Движок сайта некорректно их обрабатывает.
{: .prompt-info }

Выполним `ansible-playbook`
```bash
$ ansible-playbook -i hosts playbook2.yml

PLAY [all] *******************************************************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************************************
[WARNING]: Collection community.general does not support Ansible version 2.10.8
ok: [test-srv]

TASK [Create file] ***********************************************************************************************************************************
[WARNING]: Collection community.general does not support Ansible version 2.10.8
changed: [test-srv]

PLAY RECAP *******************************************************************************************************************************************
test-srv                     : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

В результате выполнения плэйбука появился файл, владелец которого пользователь `root`
```bash
$ ls -lh
total 20K
-rw-r--r-- 1 user user 209 Nov  8 11:58 hosts
-rw-r--r-- 1 user user 610 Nov  8 11:57 playbook.yml
-rw-r--r-- 1 user user 404 Nov  8 11:59 playbook2.yml
-rw-r--r-- 1 root root   0 Nov  8 11:59 test.txt
```

Не забываем время от времени запускать `bw sync`, чтобы подтянулась свежая копия хранилища.
