---
title: "Установка Bacula + PostgreSQL на Centos 7"
date: "2019-12-04"
categories: 
  - Backup-System
  - Database-System
tags: 
  - "backup"
  - "bacula"
  - "centos"
  - "postgresql"
image:
  path: /commons/987847_cbc9_2.jpg
  alt: "Установка Bacula"
---

> **Bacula** — кроссплатформенное клиент-серверное программное обеспечение, позволяющее управлять резервным копированием, восстановлением, и проверкой данных по сети для компьютеров и операционных систем различных типов.
{: .prompt-tip }

Добавляем репозиторий Bacula.  
Для этого переходим по [ссылке](https://www.bacula.org/bacula-binary-package-download/), заполняем форму, на почту нам скидывают доступ в репозиторий

Создаем файл с содержимым:

```sh
$ sudo cat > /etc/yum.repos.d/bacula.repo
[bacula]
name=bacula repo
baseurl=https://bacula.org/packages/%код из письма%/rpms/9.4.4/el7/$basearch/
gpgcheck=0
enabled=1
```

Далее нам надо отключить SELinux, т.к. судя по [ответу разработчиков](https://sourceforge.net/p/bacula/mailman/message/36748770/) в релизной на сегодняшней день версии `9.4.4` они еще не допилили связь Bacula и SELinux

```sh
$ sudo setenforce 0
$ sudo nano /etc/selinux/config
...
SELINUX=disabled
...
```

Открываем порты

```sh
$ sudo firewall-cmd --permanent --zone=public --add-port=9101-9103/tcp
$ sudo firewall-cmd --reload
```

## Установка и настройка PostgreSQL 10

Добавляем репозиторий PostgreSQL 10

```sh
$ sudo yum install https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm
```

Устанавливаем PostgreSQL 10, инициализируем БД сервер

```sh
$ sudo yum install postgresql10-server postgresql10-devel
$ sudo /usr/pgsql-10/bin/postgresql-10-setup initdb
Initializing database ... OK
```

Запускаем сервис и добавляем его в автозагрузку

```sh
$ sudo systemctl start postgresql-10
$ sudo systemctl enable postgresql-10
```

Создаем пользователя PostgreSQL `bacula` и задаем ему пароль `bacula`

```sh
$ sudo su - postgres
-bash-4.2$ createuser bacula
-bash-4.2$ psql
postgres=# ALTER USER bacula PASSWORD 'bacula';
postgres=# ALTER USER bacula LOGIN SUPERUSER CREATEDB CREATEROLE;
ALTER ROLE
postgres=# \q
-bash-4.2$ exit
```

Настраиваем PostgreSQL  
Раскомментируем строку `listen_addresses`

```sh
$ sudo nano /var/lib/pgsql/10/data/postgresql.conf
listen_addresses = 'localhost'
```

Настраиваем подключения к базе

```sh
$ sudo nano /var/lib/pgsql/10/data/pg_hba.conf
# TYPE  DATABASE        USER            ADDRESS                 METHOD
# "local" is for Unix domain socket connections only
local   all             all                                     peer
local   bacula          bacula                                  md5
# IPv4 local connections:
host    all             all             127.0.0.1/32            md5
```

Перезапускаем PostgreSQL

```sh
$ sudo systemctl restart postgresql-10
```

## Установка Bacula 9.4.4

Устанавливаем Bacula

```sh
$ sudo yum install bacula-postgresql
```

Выполняем скрипты, для корректной работы Bacula с PostgreSQL

```sh
$ cd /opt/bacula/scripts
$ sudo -u bacula ./create_postgresql_database
$ sudo -u bacula ./make_postgresql_tables
$ sudo -u bacula ./grant_postgresql_privileges
```

Делаем сим линки для удобства управления, запускаем службу и смотрим статус

```sh
$ sudo ln -s /opt/bacula/bin/* /usr/bin
$ sudo bacula start
$ sudo bacula status
```

Создаем директории для бэкапов / ресторов (к ним можно примонтировать NFS-хранилище)

```sh
$ sudo mkdir -p /bacula/{backup,restore}
$ sudo chown -R apache:bacula /bacula
$ sudo chmod -R 777 /bacula
```

Создаем директории для логов

```sh
$ sudo mkdir /opt/bacula/log/
$ sudo chown -R bacula:bacula /opt/bacula/log/
```

Теперь надо отредактировать конфиги Bacula, прописать наш пароль

```sh
$ sudo nano /opt/bacula/etc/bacula-dir.conf
```

### Настройка ресурса Director

Найдите ресурс Director и настройте его для прослушивания ip-адреса. Для этого добавьте в раздел строку `DirAddress`.

```
Director {
...
  Password = "bacula"
  DirAddress = 192.168.1.25
}
```

### Настройка локальных задач

Задачи Bacula (job) выполняют резервное копирование и восстановление данных. Ресурсы задания – это подробные данные о том или ином задании: имя клиента, файлы для бэкапа или восстановления (FileSet) и многое другое.  
Найдите ресурс `Job` с именем `BackupClient1`. Замените значение в строке `Name` именем `BackupLocalFiles`.

```
Job {
Name = "BackupLocalFiles"
JobDefs = "DefaultJob"
}
```

Затем найдите ресурс `Job` с именем `RestoreFiles`. В строке Name укажите имя `RestoreLocalFiles`, а в строке `Where` – каталог `/bacula/restore`.

```
Job {
  Name = "RestoreLocalFiles"
...
  Where = /bacula/restore
}
```

### Файлы для бэкапа

FileSet определяет список файлов и каталогов, которые нужно включить или исключить из резервного копирования.  
Найдите ресурс `FileSet` по имени `Full Set` (под комментарием # List of files to be backed up). Сюда нужно внести следующие изменения:

- Добавить сжатие gzip.
- В разделе Include заменить /usr/sbin в строке File на /.
- В конце раздела Exclude добавить строку File = /bacula.

```
# List of files to be backed up
FileSet {
  Name = "Full Set"
  Include {
    Options {
      signature = MD5
      compression = GZIP
    }
...
  Exclude {
  ...
    File = /bacula
  }
}
```

### Настройка каталога

Ресурс Catalog определяет БД, к которой будет подключаться Director.  
Найдите ресурс `Catalog` по имени `MyCatalog` (под комментарием Generic catalog service). Обновите значение `dbpassword` и укажите пароль пользователя MySQL/PostgeSQL `bacula`.  
Еще надо добавить адрес, иначе при проверке (sudo bacula-dir -tc /opt/bacula/etc/bacula-dir.conf) нет коннекта к базе

```
# Generic catalog service
Catalog {
  Name = MyCatalog
  dbname = "bacula"; DB Address = "localhost"; dbuser = "bacula"; dbpassword = "bacula"
}
```

### Настройка пула

Ресурс Pool определяет набор хранилищ, используемых Bacula для записи резервных копий. В данном случае в качестве томов хранения используются файлы. Обновите метки, чтобы локальные резервные копии были правильно помечены.

```
# File Pool definition
Pool {
  ...
  Label Format = "Local-"               # Auto label
}
```

Сохраните и закройте файл. Настройка компонента Bacula Director завершена.

Проверка настроек

```sh
$ sudo bacula-dir -tc /opt/bacula/etc/bacula-dir.conf
```

### Настройка Storage Daemon

Надо настроить Storage Daemon, чтобы система Bacula понимала, где хранить файлы.

Откройте конфигурационный файл SD.

```sh
$ sudo nano /opt/bacula/etc/bacula-sd.conf
Storage {                             # definition of myself
  ...
  SDAddress = 192.168.1.25
}
```

### Настройка устройства хранения

```
Autochanger {
  Name = FileChgr1
  Device = FileChgr1-Dev1, FileChgr1-Dev2
  Changer Command = ""
  Changer Device = /dev/null
}

Device {
  Name = FileChgr1-Dev1
  Media Type = File1
  Archive Device = /bacula/backup
  ...
}

Device {
  Name = FileChgr1-Dev2
  Media Type = File1
  Archive Device = /bacula/backup
  ...
}
```

Проверка настроек

```sh
$ sudo bacula-sd -tc /opt/bacula/etc/bacula-sd.conf
```

Во всех файлах надо поменять пароль на наш `bacula`
