---
title: "Установка LAMP (Linux, Apache, MySQL, PHP)"
date: "2015-07-28"
categories: 
  - Web
tags: 
  - "apache"
  - "centos"
  - "iptables"
  - "linux"
  - "mysql"
  - "php"
image:
  path: /commons/virtual-reality-development.jpg
  alt: "Установка LAMP"
---

> **LAMP** - это комплекс серверного программного обеспечения, широко используемый в сети Интернет. Он состоит из четырёх компонентов:
> - Linux - операционная система для сервера
> - Apache - веб-сервер для обслуживания веб-приложений
> - MySQL - реляционная система управления базами данных для хранения и управления данными
> - PHP - скриптовый язык для разработки динамических веб-приложений
{: .prompt-tip }

Добавляем репозиторий Atomic

```sh
$ wget -q -O - http://www.atomicorp.com/installers/atomic.sh | sh
$ sudo ./atomic.sh
```

Устанавливаем пакеты

```sh
$ sudo yum -y install httpd php mysql mysql-server php-mysql php-pear php-pdo php-pgsql php-pecl-memcache php-gd php-mbstring php-mcrypt php-xml
```

Запускаем MySQL

```sh
$ sudo service mysqld start
```

Задаем root-пароль для MySQL

```sh
$ mysqladmin -u root password 'ENTER-PASSWORD-HERE'
```

Коннектимся к MySQL и удаляем тестовую базу

```sh
$ mysql -u root -p
=> DROP DATABASE test;
=> DELETE FROM mysql.user WHERE user = '';
=> FLUSH PRIVILEGES;
```

Прописываем сервисы в автозапуск и запускаем их

```sh
$ sudo chkconfig httpd on
$ sudo chkconfig mysqld on
$ sudo service httpd start
$ sudo service mysqld start
```

Открываем 80-порт снаружи

```sh
$ sudo nano /etc/sysconfig/iptables
```

Прописываем

```
-A INPUT -p tcp -m state --state NEW -m tcp --dport 80 -j ACCEPT
```

Проверка. Создаем файл `phpinfo.php`

```sh
$ sudo nano /var/www/html/phpinfo.php
```

Прописываем в него:
```
<?php echo phpinfo(); ?>
```

Проверяем работоспособность, открыв страницу `phpinfo.php` в браузере.
