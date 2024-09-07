---
title: "Меняем стандартный SSH порт в Centos 7"
date: "2019-10-23"
categories: 
  - Manuals
tags: 
  - "centos"
  - "ssh"
  - "selinux"
image:
  path: /commons/bigstock-Terminal-startup-icon-direct-90816239_6.jpg
  alt: "Меняем стандартный SSH порт"
---

> **SSH** — сетевой протокол прикладного уровня, позволяющий производить удалённое управление операционной системой и туннелирование TCP-соединений.
{: .prompt-tip }

Проверяем, что разрешено на сервере в firewall

```sh
$ sudo firewall-cmd --permanent --list-all
public (default)
  interfaces:
  sources:
  services: ssh dhcpv6-client
  masquerade: no
  forward-ports:
  icmp-blocks:
  rich rules:
```

Открываем порт, на который мы хотим повесить OpenSSH

```sh
$ sudo firewall-cmd --permanent --zone=public --add-port=6622/tcp
$ sudo firewall-cmd --reload
```

Проверяем

```sh
$ sudo firewall-cmd --zone=public --list-ports
6622/tcp
```

Редактируем конфигурационный файл `sshd_config`

```sh
$ sudo nano /etc/ssh/sshd_config
...
Port 6622
...
```

Тем самым мы поменяли стандартный порт 22 на 6622.

> На данном этапе **НЕ СЛЕДУЕТ** перезапускать службу SSH
{: .prompt-warning }

Если вы не отключали SELinux, надо внести некоторые изменения

```sh
$ sudo yum install policycoreutils-python
$ sudo semanage port -a -t ssh_port_t -p tcp 6622
```

Вот теперь можно перезагружать службу `sshd`

```sh
$ sudo systemctl restart sshd
```

Проверяем подключение по ssh на новом порту 6622. 
Если все ок, закрываем доступ к стандартному порту

```sh
$ sudo firewall-cmd --permanent --zone=public --remove-service=ssh
$ sudo firewall-cmd --reload
```
