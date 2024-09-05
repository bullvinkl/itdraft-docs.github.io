---
title: "Установка и обновление Zabbix-agent в CentOS 7"
date: "2017-10-10"
categories: 
  - Linux
  - Zabbix
tags: 
  - "centos"
  - "zabbix"
image:
  path: /commons/computer3.jpg
  alt: "Установка и обновление Zabbix-agent"
---

> **Zabbix-agent** - это программное обеспечение, предназначенное для мониторинга и сбора метрик о работе локальных ресурсов, таких как серверы, сетевое оборудование, приложения и т.д. Агент является частью системы мониторинга Zabbix

## Установка zabbix-agent

Добавляем репозиторий Zabbix (версия 3.2)

```sh
$ sudo rpm -ivh https://repo.zabbix.com/zabbix/3.2/rhel/7/x86_64/zabbix-release-3.2-1.el7.noarch.rpm
```

Обновляемся

```sh
$ sudo yum update
```

Ставим zabbix-agent:

```sh
$ sudo yum install zabbix-agent
```

Редактируем конфигурационный файл:

```sh
$ sudo nano /etc/zabbix/zabbix_agentd.conf
```

Прописываем ip-адрес zabbix-сервера и имя нашей машины, которую будем мониторить

```
Server=192.168.1.37
Hostname=srv-serv-01
```

Настройка файерволла:

```sh
$ sudo firewall-cmd --permanent --new-service=zabbix
$ sudo firewall-cmd --permanent --service=zabbix --add-port=10050/tcp
$ sudo firewall-cmd --permanent --service=zabbix --set-short="Zabbix Agent"
$ sudo firewall-cmd --permanent --add-service=zabbix
```

Перезапускаем файерволл:

```sh
$ sudo firewall-cmd --reload
```

Добавляем zabbix-agent в автозагрузку, запускаем его и проверяем статус

```sh
$ sudo systemctl enable zabbix-agent
$ sudo systemctl start zabbix-agent
$ sudo systemctl status zabbix-agent
```

## Обновление zabbix-agent

Обновляем zabbix-agent с версии 3.4 до версии 3.4

```sh
$ sudo rpm -Uvh https://repo.zabbix.com/zabbix/3.4/rhel/7/x86_64/zabbix-release-3.4-2.el7.noarch.rpm
Загружается https://repo.zabbix.com/zabbix/3.4/rhel/7/x86_64/zabbix-release-3.4-2.el7.noarch.rpm
Подготовка...               ################################# [100%]
Обновление / установка...
   1:zabbix-release-3.4-2.el7         ################################# [ 50%]
Очистка / удаление... 
   2:zabbix-release-3.2-1.el7         ################################# [100%]
```

Обновляемся

```sh
$ sudo yum update
```