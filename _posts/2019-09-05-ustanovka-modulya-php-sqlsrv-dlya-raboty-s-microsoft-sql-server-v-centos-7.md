---
title: "Установка модуля PHP-SQLSRV для работы с Microsoft SQL Server в Centos 7"
date: "2019-09-05"
categories: 
  - Web
tags: 
  - "centos"
  - "php-sqlsrv"
image:
  path: /commons/7592.png
  alt: "PHP-SQLSRV для работы с Microsoft SQL Server в Centos"
---

> **PDO_SQLSRV** - это драйвер, реализующий интерфейс PHP Data Objects (PDO) для получения доступа из PHP к базам данных MS SQL Server (начиная с версии SQL Server 2005) и SQL Azure.
{: .prompt-tip }

[Установка PHP 7.x]({% post_url 2018-07-27-ustanovka-php-7-na-centos-7 %}) в Centos 7 была рассмотрена раньше

## Установка необходимых компонентов

Добавляем репозиторий

```sh
$ sudo curl https://packages.microsoft.com/config/rhel/7/prod.repo > /etc/yum.repos.d/mssql-release.repo
```

Удаляем старые компоненты `unixODBC` (если они были установлены)

```sh
$ sudo yum remove unixODBC-utf16 unixODBC-utf16-devel
```

Устанавливаем новые

```sh
$ sudo ACCEPT_EULA=Y yum install msodbcsql17
$ sudo ACCEPT_EULA=Y yum install mssql-tools
```

Добавляем переменные среды в профиль, для указания оболочке, где искать исполняемые файлы

```sh
$ sudo echo 'export PATH="$PATH:/opt/mssql-tools/bin"' >> ~/.bash_profile
$ sudo echo 'export PATH="$PATH:/opt/mssql-tools/bin"' >> ~/.bashrc
$ sudo source ~/.bashrc
```

Устанавливаем `unixODBC-devel`

```sh
$ sudo yum install unixODBC-devel
```

Устанавливаем `php-sqlsrv`

```sh
$ sudo yum install php-sqlsrv
```

Теперь осталось перезапустить Apache / Nginx

```sh
$ sudo systemctl restart nginx
$ sudo systemctl restart httpd
```
