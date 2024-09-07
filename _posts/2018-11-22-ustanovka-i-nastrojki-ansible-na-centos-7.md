---
title: "Установка и настройки Ansible на CentOS 7"
date: "2018-11-22"
categories: 
  - Linux
  - Ansible
tags: 
  - ansible
  - ssh-copy-id
  - sshpass
image:
  path: /commons/pexels-christina-morillo-1181244-scaled.jpg
  alt: "Установка и настройки Ansible"
---

> **Ansible** — система управления конфигурациями, написанная на Python, с использованием декларативного языка разметки для описания конфигураций. Используется для автоматизации настройки и развертывания программного обеспечения. Обычно используется для управления Linux-узлами, но Windows также поддерживается. Поддерживает работу с сетевыми устройствами, на которых установлен Python версии 2.4 и выше по SSH или WinRM соединению.
{: .prompt-tip }

## Установка Ansible

Добавляем в систему репозиторий EPEL и устанавливаем программу

```sh
$ sudo yum install epel-release
$ sudo yum install ansible
```

Настройки удалённых хостов хранятся в файле `/etc/ansible/hosts`  
Сделаем резервную копию этого файла

```sh
$ sudo mv /etc/ansible/hosts /etc/ansible/hosts.default
```

## Настройка подключения

На сервере с Ansible сгенерируем RSA-ключ

```sh
$ sudo ssh-keygen -t rsa
```

На удаленном сервере (который будем администрировать с помощью Ansible) включаем в SSH авторизацию  по ключам.  
Для этого редактируем конфигурационный файл `sshd_config` и перезапускаем службу

```sh
$ sudo nano /etc/ssh/sshd_config
PubkeyAuthentication yes 
AuthorizedKeysFile .ssh/authorized_keys

$ sudo service sshd restart
```

На сервере c Ansible копируем `id_pub.pub` на удаленный сервер с помощью утилиты `ssh-copy-id`

```sh
$ sudo ssh-copy-id -i ~/.ssh/id_rsa.pub root@192.168.1.84
```

Тестируем подключение, при этом пароль не должен запрашиваться

```sh
$ sudo ssh 192.168.1.84
```

Добавим наш тестовый сервер в файл `hosts`, для дальнейшего тестирования возможностей Ansible

```sh
$ sudo nano /etc/ansible/hosts
[test]
centos ansible_ssh_host=192.168.1.86
```

Проверяем работу Ansible

```sh
$ sudo ansible test -m ping
centos | SUCCESS => {
    "changed": false, 
    "ping": "pong"
}
```

где `test` - группа ПК (см. файл `/etc/ansible/hosts`)

Если нет желания включать авторизацию в SSH по файлу, можно установить утилиту `sshpass`, и посмотреть на ответ Ansible

```sh
$ sudo yum install sshpass
$ sudo ansible test -m ping -k -u root
SSH password:
centos | SUCCESS => {
    "changed": false, 
    "ping": "pong"
}
```

## Некоторые команды

```sh
$ ansible -m ping all    - пинг
$ ansible test -m shell -a 'cat /etc/redhat-release'     - Версия Centos
$ ansible test -m shell -a 'uname -a'     - Выполнить команду uname -a
$ ansible test  -m setup - узнать информацию о сервере
$ ansible -i hosts -m setup -a 'filter=ansible_memtotal_mb' all      - сколько памяти доступно на всех хостах
$ ansible -i hosts -m setup -a 'filter=ansible_all_ipv4_addresses' all      - узнать ip-адреса всех хостов
$ ansible -i hosts -m setup -a 'filter=ansible_default_ipv4' all
$ ansible -i hosts -m setup -a 'filter=ansible_distribution' all
$ ansible -i hosts -m setup -a 'filter=ansible_distribution_version' all
$ ansible -i hosts -m setup -a 'filter=ansible_memory_mb' all
$ ansible -i hosts -m setup -a 'filter=ansible_processor' all
```
