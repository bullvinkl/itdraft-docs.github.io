---
title: "Установка и настройка FTP-сервера VSFTP на Centos 7. Локальные пользователи"
date: "2018-07-23"
categories: 
  - Storage-System
tags: 
  - "bash"
  - "centos"
  - "vsftpd"
image:
  path: /commons/bigstock-laptop.jpg
  alt: "Установка и настройка FTP-сервера VSFTP"
---

> **VSFTP** (Very Secure FTP Daemon) - это сервер FTP с высокими требованиями к безопасности, разработанный для операционных систем, подобных UNIX. Он обеспечивает безопасную передачу файлов между клиентом и сервером, защищает пароли и данные от несанкционированного доступа.
{: .prompt-tip }

## Установка FTP-сервера

Устанавливаем софт:

```sh
$ sudo yum install vsftpd nano net-tools -y
```

Создаем директорию, где будут каталоги пользователей и выставляем права доступа

```sh
$ sudo mkdir /home/vsftpd
$ sudo chmod 0777 /home/vsftpd
```

Cохраняем дефолтный конфиг

```sh
$ sudo mv /etc/vsftpd/vsftpd.conf /etc/vsftpd/vsftpd.conf_backup
```

Пишем свой конфиг

```sh
$ sudo nano /etc/vsftpd/vsftpd.conf
```

```sh
# Запуск сервера в режиме службы
listen=YES

# Работа в фоновом режиме
background=YES

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
userlist_enable=YES

# Файл со списками разрешенных к подключению пользователей
userlist_file=/etc/vsftpd/user_list

# Пользователь будет отклонен, если его нет в user_list
userlist_deny=NO

# Директория с настройками пользователей
user_config_dir=/etc/vsftpd/users

# Показывать файлы, начинающиеся с точки
force_dot_files=YES

# Маска прав доступа к создаваемым файлам
local_umask=022

# Порты для пассивного режима работы
pasv_min_port=49000
pasv_max_port=55000
```

Добавляем пользователя

```sh
$ sudo useradd -s /sbin/nologin ftpuser
$ sudo passwd ftpuser
fptpassword
```

Создаем папку, где будут отдельные конфиги пользователей

```sh
$ sudo mkdir /etc/vsftpd/users
$ sudo touch /etc/vsftpd/users/ftpuser
```

Задаем в конфиге домашний ftp-каталог

```sh
$echo 'local_root=/home/vsftpd/ftpuser/' | sudo tee -a /etc/vsftpd/users/ftpuser
```

Cоздаем каталог пользователя и задаем владельца

```sh
$ sudo mkdir /home/vsftpd/ftpuser
$ sudo chown ftpuser:ftpuser /home/vsftpd/ftpuser
```

Создаем файл, в котором будет перечислен список пользователей, которым разрешен выход из домашнего каталога. И добавляем в него пользователя root

```sh
$ sudo touch /etc/vsftpd/chroot_list
$ echo 'root' | sudo tee -a /etc/vsftpd/chroot_list
```

Создаем файл, в котором будет перечислен список пользователей,  которым разрешено подключаться к FTP-серверу. Добавляем в него пользователей root и ftpuser

```sh
$ sudo touch /etc/vsftpd/user_list
$ echo 'root' | sudo tee -a /etc/vsftpd/user_list
$ echo 'ftpuser' | sudo tee -a /etc/vsftpd/user_list
```

Создаем файл, куда будут писаться логи, и выставляем на него права

```sh
$ sudo touch /var/log/vsftpd.log
$ sudo chmod 600 /var/log/vsftpd.log
```

Добавляем сервис vsftpd в автозагрузку, запускаем его, проверяем статус

```sh
$ sudo systemctl enable vsftpd
$ sudo systemctl start vsftpd
$ sudo systemctl status vsftpd
```

Смотрим, появился ли процесс

```sh
$ sudo netstat -tulnp | grep vsftpd
	tcp 0 0 0.0.0.0:21 0.0.0.0:* LISTEN 13195/vsftpd
```

Добавляем правила в firewall: открываем порты 21, 49000-55000

```sh
$ sudo firewall-cmd --permanent --add-port=21/tcp
$ sudo firewall-cmd --permanent --add-port=49000-55000/tcp
$ sudo firewall-cmd --reload
```

Отключаем Selinux, перезагружаемся

```sh
$ sudo setenforce 0
$ sudo nano /etc/selinux/config
	SELINUX=disabled
$ sudo reboot
```

## Инструкция по добавлению новых пользователей

Добавляем пользователя в систему

```sh
$ sudo useradd -s /sbin/nologin testuser
$ sudo passwd testuser
testpassword
```

Создаем каталог пользователя и задаем владельца

```sh
$ sudo mkdir /home/vsftpd/testuser
$ sudo chown test:test /home/vsftpd/testuser
```

Создаем папку, где будут отдельные конфиги пользователей

```sh
$ sudo touch /etc/vsftpd/users/testuser
```

Задаем в конфиге домашний фтп-каталог

```sh
$ echo 'local_root=/home/vsftpd/testuser/' | sudo tee -a /etc/vsftpd/users/testuser
```

Задаем разрешенных пользователей

```sh
$ echo 'testuser' | sudo tee -a /etc/vsftpd/user_list
```

Перезапускаем vsftpd

```sh
$ sudo systemctl restart vsftpd
```

## UPD 27.11.2018  
Bash-скрипт добавления пользователей

Создаем bash-скрипт `add_ftp_user.sh`

```sh
$ sudo nano /home/add_ftp_user.sh 
#!/bin/bash

NAME=$1
PASS=$2

echo "USAGE: add_ftp_user.sh [username] [password]"

# проверка входных параметров
if [ -z "$NAME" ]; then
    echo "Error: username is not set"
    exit
fi

if [ -z "$PASS" ]; then
    echo "Error: password not set"
    exit
fi

# создаем системных пользователей
echo "Creating user: $NAME"
echo "With password: $PASS"

useradd -s /sbin/nologin -p `openssl passwd -1 $PASS` $NAME

# сохраняем данные в файл /etc/vsftpd/new_ftp_users_list
echo "user: $NAME, pass: $PASS" >> /etc/vsftpd/new_ftp_users_list

# создаем ftp-директорию пользователя
mkdir /home/vsftpd/$NAME

# назначаем владельца
chown $NAME:$NAME /home/vsftpd/$NAME

# создаем пустой конфигурационный файл
touch /etc/vsftpd/users/$NAME

# прописываем домашний каталог
echo "local_root=/home/vsftpd/$NAME/" >> /etc/vsftpd/users/$NAME

# добавляем пользователя в список разрешенных для подключения
echo "$NAME" >> /etc/vsftpd/user_list

# задаем права каталога пользователя
chmod 0777 /home/vsftpd/$NAME

# перезапускаем службу vsftp
systemctl restart vsftpd
```

Делаем его исполняемым

```sh
$ sudo chmod +x /home/add_ftp_user.sh 
```

теперь, что бы добавить нового FTP-пользователя надо выполнить команду

```sh
$ сd /home
$ sudo ./add_ftp_user.sh %user% %pass%
```

где  
- `%user%` - логин  
- `%pass%` - пароль

## UPD 03.09.2019 - не работает авторизация, ошибка 530

Не работает авторизация в vsftpd:

```
530 Login incorrect
```

**Решение:**  
Данная ошибка получается из-за того, что в файле **/etc/pam.d/vsftpd**  
присутствует строчка:

```sh
$ sudo egrep -v "^#|^$" /etc/pam.d/vsftpd
...
auth       required     pam_shells.so
...
```

она означает, что только пользователям с доступом к оболочкам должен быть разрешен доступ  
А добавляли мы пользователя как раз с параметром: **\-s /sbin/nologin**

```sh
$ sudo useradd -s /sbin/nologin ftpuser
```

Комментирует эту строчку: auth required pam\_shells.so и перезапускаем vsftpd

```sh
$ sudo systemctl restart vsftpd
```
