---
title: "FTP-сервер VSFTP и virtual users MySQL на CentOS 7, Web-админка для VSFTP"
date: "2018-07-27"
categories: 
  - Linux
tags: 
  - "centos"
  - "mysql"
  - "vsftpd"
image:
  path: /commons/computer2.jpg
  alt: "FTP-сервер VSFTP и virtual users"
---

> **VSFTP** (Very Secure FTP Daemon) - это сервер FTP с высокими требованиями к безопасности, разработанный для операционных систем, подобных UNIX. Он обеспечивает безопасную передачу файлов между клиентом и сервером, защищает пароли и данные от несанкционированного доступа.
{: .prompt-tip }

## Установка Apache, PHP, MySQL

Добавляем репозиторий EPEL и обновляемся

```sh
$ sudo yum install epel-release
$ sudo yum update
```

Ставим софт для удобства работы

```sh
$ sudo yum install nano htop mc wget
```

Устанавливаем MySQL-сервер MariaDB, Web-сервер Apache и PHP. Инструкцию по установке можно прочитать по ссылкам ниже:

- [Установка Web-сервера Apache в Centos 7](https://itdraft.ru/2018/07/27/ustanovka-web-servera-apache-na-centos-7/)

- [Установка MySQL-сервера (MariaDB) в Centos 7](https://itdraft.ru/2018/07/27/ustanovka-mysql-servera-mariadb-na-centos-7/)

- [Установка PHP 7 в Centos 7](https://itdraft.ru/2018/07/27/ustanovka-php-7-na-centos-7/)

Перезапускаем Apache

```sh
$ sudo systemctl restart httpd.service
```

## Отключение Selinux

Отключаем SElinux

```sh
$ sudo setenforce 0
$ sudo nano /etc/selinux/config
...
SELINUX=disabled
...
```

## Настройка Apache

Настраиваем Apache, добавим `vhosts` - несколько сайтов на одном ip-адресе

```sh
$ sudo nano /etc/httpd/conf.d/vhosts.conf
...
# Загрузка моих vhosts
IncludeOptional vhosts.d/*.conf
```

Создаем каталог, где будут лежать конфигурации `vhosts`

```sh
$ sudo mkdir /etc/httpd/vhosts.d
```

Создаем конфигурационный файл для локального IP

```sh
$ sudo nano /etc/httpd/vhosts.d/192.168.1.19.conf
<VirtualHost 192.168.1.19:80>
  ServerAdmin admin@itdraft.ru
  DocumentRoot "/var/www/192.168.1.19/html/"
  ServerName 192.168.1.19
  ErrorLog "/var/www/192.168.1.19/logs/error_log"
  CustomLog "/var/www/192.168.1.19/logs/access_log" combined

  <Directory "/var/www/192.168.1.19/html/">
  DirectoryIndex index.html index.php
  Options FollowSymLinks
  AllowOverride All
  Require all granted
  </Directory>
</VirtualHost>
```

Создаем необходимые каталоги и выставляем права доступа

```sh
$ sudo mkdir -p /var/www/192.168.1.19/html
$ sudo mkdir -p /var/www/192.168.1.19/logs
$ sudo chown -R apache:apache /var/www/192.168.1.19/
```

Перезапускаем Apache

```sh
$ sudo systemctl restart httpd.service
```

## Установка VSFTP

Устанавливаем FTP-сервер

```sh
$ sudo yum install vsftpd
```

Редактируем конфигурационный файл

```sh
$ sudo nano /etc/vsftpd/vsftpd.conf
# Запуск сервера в режиме службы
listen=YES

# Работа в фоновом режиме
background=YES

# Разрешить подключаться виртуальным пользователям
guest_enable=YES

# Системный пользователь от имени котрого подключаются виртуальные
guest_username=apache

# Виртуальные пользователи имеют те же привелегии, что и локальные
virtual_use_local_privs=YES

# Автоматическое назначение домашнего каталога для виртуальных пользователей
user_sub_token=$USER
local_root=/home/ftp/$USER

# Имя pam сервиса для vsftpd
pam_service_name=vsftpd

# Входящие соединения контроллируются через tcp_wrappers
tcp_wrappers=YES

# Запрещает подключение анонимных пользователей
anonymous_enable=NO

# Каталог, куда будут попадать анонимные пользователи, если они разрешены
#anon_root=/ftp

# Разрешает вход для локальных пользователей
local_enable=YES

# Разрешены команды на запись и изменение
write_enable=YES

# Указывает исходящим с сервера соединениям использовать 20-й порт
connect_from_port_20=YES

# Логирование всех действий на сервере
xferlog_enable=YES

# Путь к лог-файлу
xferlog_file=/var/log/vsftpd.log

# Включение специальных ftp команд, некоторые клиенты без этого могут зависать
async_abor_enable=YES

# Локальные пользователи по-умолчанию не могут выходить за пределы своего домашнего каталога
chroot_local_user=YES

# Разрешить список пользователей, которые могут выходить за пределы домашнего каталога
chroot_list_enable=YES

# Список пользователей, которым разрешен выход из домашнего каталога
chroot_list_file=/etc/vsftpd/chroot_list

# Разрешить запись в корень chroot каталога пользователя
allow_writeable_chroot=YES

# Контроль доступа к серверу через отдельный список пользователей
#userlist_enable=YES

# Файл со списками разрешенных к подключению пользователей
#userlist_file=/etc/vsftpd/user_list

# Пользователь будет отклонен, если его нет в user_list
#userlist_deny=NO

# Директория с настройками пользователей
user_config_dir=/etc/vsftpd/users

# Показывать файлы, начинающиеся с точки
force_dot_files=YES

# Маска прав доступа к создаваемым файлам
local_umask=022

# Приветствие
ftpd_banner=Welcome to FTP service.

#set maximum allowed connections per single IP address (0 = no limits)
max_per_ip=10

# Порты для пассивного режима работы
pasv_min_port=40900
pasv_max_port=40999

data_connection_timeout=900
idle_session_timeout=900
```

Открываем порты 20-21 (активный режим работы ftp-сервера) и 40900-40999 (пассивный режим работы ftp-сервера) 

```sh
$ sudo firewall-cmd --permanent --add-port=20-21/tcp
$ sudo firewall-cmd --permanent --add-port=40900-40999/tcp
$ sudo firewall-cmd --reload
```

Добавляем сервер в автозагрузку, запускаем его и проверяем статус

```sh
$ sudo systemctl enable vsftpd
$ sudo systemctl start vsftpd
$ sudo systemctl status vsftpd
```

## Настройка MySQL

Подключаемся к MySQL

```sh
mysql -u root -p
```

Создаем базу данных и пользователя с правами только на эту базу

```sh
> CREATE DATABASE vsftpd;
> GRANT SELECT ON vsftpd.* TO 'vsftpd'@'localhost' IDENTIFIED BY 'passwordftp';
> USE vsftpd;
```

Выбираем эту базу и создаем таблицу. Структура таблицы задана с учетом использования функционала web-админки

```sh
> USE vsftpd;
> CREATE TABLE `users` (
`id` INT NOT NULL AUTO_INCREMENT PRIMARY KEY ,
`username` VARCHAR( 30 ) CHARACTER SET utf8 COLLATE utf8_unicode_ci NOT NULL ,
`password` VARCHAR( 50 ) CHARACTER SET utf8 COLLATE utf8_unicode_ci NOT NULL ,
`email` varchar(80) CHARACTER SET utf8 NOT NULL,
`date` datetime NOT NULL,
`date_end` date NOT NULL,
`temp` int(1) NOT NULL,
`dir` varchar(80) CHARACTER SET utf8 COLLATE utf8_unicode_ci NOT NULL,
`local_ip` varchar(30) CHARACTER SET utf8 NOT NULL,
UNIQUE (`username`)
) ENGINE = MYISAM ;
```

Мой dump базы данных

```sh
-- Хост: localhost
-- Время создания: Июл 27 2018 г., 12:08
-- Версия сервера: 5.5.56-MariaDB
-- Версия PHP: 7.2.3

SET SQL_MODE = "NO_AUTO_VALUE_ON_ZERO";
SET AUTOCOMMIT = 0;
START TRANSACTION;
SET time_zone = "+00:00";


/*!40101 SET @OLD_CHARACTER_SET_CLIENT=@@CHARACTER_SET_CLIENT */;
/*!40101 SET @OLD_CHARACTER_SET_RESULTS=@@CHARACTER_SET_RESULTS */;
/*!40101 SET @OLD_COLLATION_CONNECTION=@@COLLATION_CONNECTION */;
/*!40101 SET NAMES utf8mb4 */;

--
-- База данных: `vsftpd`
--

-- --------------------------------------------------------

--
-- Структура таблицы `users`
--

CREATE TABLE `users` (
  `id` int(11) NOT NULL,
  `username` varchar(30) CHARACTER SET utf8 COLLATE utf8_unicode_ci NOT NULL,
  `password` varchar(50) CHARACTER SET utf8 COLLATE utf8_unicode_ci NOT NULL,
  `email` varchar(80) CHARACTER SET utf8 NOT NULL,
  `date` datetime NOT NULL,
  `date_end` date NOT NULL,
  `temp` int(1) NOT NULL,
  `dir` varchar(80) CHARACTER SET utf8 COLLATE utf8_unicode_ci NOT NULL,
  `local_ip` varchar(30) CHARACTER SET utf8 NOT NULL
) ENGINE=MyISAM DEFAULT CHARSET=latin1;

--
-- Дамп данных таблицы `users`
--

INSERT INTO `users` (`id`, `username`, `password`, `email`, `date`, `date_end`, `temp`, `dir`, `local_ip`) VALUES
(104, 'user', md5('password'), 'admin@itdraft.ru', '2018-03-12 13:39:24', '2018-04-12', 0, '/home/ftp/user', '');

--
-- Индексы сохранённых таблиц
--

--
-- Индексы таблицы `users`
--
ALTER TABLE `users`
  ADD PRIMARY KEY (`id`),
  ADD UNIQUE KEY `username` (`username`);

--
-- AUTO_INCREMENT для сохранённых таблиц
--

--
-- AUTO_INCREMENT для таблицы `users`
--
ALTER TABLE `users`
  MODIFY `id` int(11) NOT NULL AUTO_INCREMENT, AUTO_INCREMENT=143;
COMMIT;
```

Выходим из MySQL

```sh
> \q
```

Устанавливаем модуль `pam_mysql`

```sh
$ sudo rpm -Uvh ftp://ftp.pbone.net/mirror/archive.fedoraproject.org/fedora/linux/releases/20/Everything/x86_64/os/Packages/p/pam_mysql-0.7-0.16.rc1.fc20.x86_64.rpm
```

Сохраняем копию `pam` файла

```sh
$ sudo mv /etc/pam.d/vsftpd /etc/pam.d/vsftpd.back
```

Создаем новый `pam` файл

```sh
$ sudo nano /etc/pam.d/vsftpd
session optional pam_keyinit.so force revoke
auth required pam_mysql.so user=vsftpd_user passwd=passwordftp host=localhost db=vsftpd table=users usercolumn=username passwdcolumn=password crypt=3
account required pam_mysql.so user=vsftpd_user passwd=passwordftp host=localhost db=vsftpd table=users usercolumn=username passwdcolumn=password crypt=3
```

где
- `user=vsftpd_user` - логин ароль для подключения к базе данных;
- `passwd=passwordftp` - пароль для подключения к базе данных;
- `db=vsftpd` - имя созданной базы данных;
- `table=users` - таблица с пользователями;
- `usercolumn=username` - название колонки, из которого извлекаем логин;
- `passwdcolumn=password` - название колонки, из которого извлекаем пароль;

Создаем файл с логами для `vsftpd` и выставляем на него права

```sh
$ sudo touch /var/log/vsftpd.log && chmod 600 /var/log/vsftpd.log
```

Создаем файл, в котором будет перечислен список пользователей, которым разрешен выход из домашнего каталога. И добавляем в него пользователя `root`

```sh
$ sudo touch /etc/vsftpd/chroot_list
$ echo 'root' | sudo tee -a /etc/vsftpd/chroot_list
```

Задаем пользователя и группу для директории, где будут хранится каталоги FTP-пользователей

```sh
$ sudo chown -R apache:apache /home/ftp
```

## Web-Админка для VSFTP

Web админку можно скачать по ссылке ниже

[Скачать (Github)](https://github.com/bullvinkl/ftpadmin)

Основной функционал админки:

- Добавить FTP-пользователя 
- Удалить FTP-пользователя
- Поменять пароль FTP-пользователю

Дополнительный функционал админки:

- Пользователь может быть постоянный или временный. Временный автоматически удаляется через месяц
- Чтобы продлить срок действия аккаунта временного пользователя, надо обновить ему пароль, тогда аккаунт будет действовать еще месяц.
- При добавлении пользователя пароль генерируется автоматически, его можно перегенерировать, но вручную пароль вводить нельзя
- При добавлении пользователя вы можете указать e-mail. На него будет высылаться вся информация (логин/пароль при первом добавлении, новый пароль при изменении, предупреждение об удалении временного аккаунта за день до удаления, информирование об окончательном удалении временного аккаунта)
- Такие-же письма высылаются администратору Web-админки (e-mail задается в php)

## Демонстрация возможностей web-интерфейса для VSFTP

[Youtube](https://www.youtube.com/watch?v=hosfClK-VrU)
