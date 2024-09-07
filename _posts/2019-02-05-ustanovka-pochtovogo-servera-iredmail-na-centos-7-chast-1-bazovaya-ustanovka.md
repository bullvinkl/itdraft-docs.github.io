---
title: "Установка почтового сервера iRedMail на CentOS 7. Часть 1. Базовая установка"
date: "2019-02-05"
categories: 
  - Linux
  - iRedMail
tags: 
  - "centos"
  - "iredmail"
image:
  path: /commons/ai_images-min.png
  alt: "Установка iRedMail"
---

> **iRedMail** - это бесплатное почтовое решение на базе открытого ПО, предназначенное для дистрибутивов Linux. Это полноценный почтовый сервер, который включает в себя Postfix и Dovecot, а также ряд дополнительных служб и настроек, разработанных для упрощения администрирования и обеспечения безопасности.
{: .prompt-tip }

## Подготовительный этап

Обновляемся, добавляем репозиторий EPEL, устанавливаем необходимый софт

```sh
$ sudo yum update -y
$ sudo yum install epel-release -y
$ sudo yum install htop nano mc zip unzip wget -y
```

Смотрим какой у нас сейчас hostname

```sh
$ hostname -f
srv-mail-01

$ hostname -s
srv-mail-01
```

Меняем hostname на необходимый для почтового сервера

```sh
$ sudo hostnamectl set-hostname mail.itdraft.ru
$ hostname -f
mail.itdraft.ru

$ hostname -s
mail
```

Приводим файл /etc/hosts к следующему виду

```sh
$ sudo nano /etc/hosts
127.0.0.1    localhost
%ip%         mail.itdraft.ru mail
```

где

- `%ip%` - ваш внешний ip-адрес

Отключаем SELinux и перезагружаемся

```sh
$ sudo setenforce 0
$ sudo nano /etc/selinux/config
SELINUX=disabled
$ sudo reboot
```

## Установка iRedMail

Скачиваем свежую версию iRedMail с официального сайта

```sh
$ cd
$ wget https://bitbucket.org/zhb/iredmail/downloads/iRedMail-0.9.9.tar.bz2
```

Устанавливаем пакет `bzip2` и распаковываем скаченный архив

```sh
$ sudo yum install bzip2 -y
$ tar xjf iRedMail-0.9.9.tar.bz2
```

Переходим в каталог, куда мы распаковали `iRedMail` и делаем установочный скрипт исполняемым

```sh
$ cd ~/iRedMail-0.9.9
$ sudo chmod +x iRedMail.sh
```

Запускаем установку iRedMail

```sh
$ sudo ./iRedMail.sh
```

- В открывшемся приветствии отвечаем: `Yes`
- Вводим путь для хранения почты: `/var/vmail`
- Выбираем веб-сервер: `Nginx`
- Выбираем СУБД: `MariaDB`
- Задаем пароль администратора базы данных: 
- Вводим наш почтовый домен: `itdraft.ru`
- Задаем пароль для администратора почтовыми ящиками (postmaster@itdraft.ru):
- Выбираем дополнения для удобства работы с iRedMail: `все`
- Подтверждаем введенные настройки `y` и нажимаем `Enter`. На все последующие вопросы тоже отвечаем `y`
- Ждем окончания процесса
