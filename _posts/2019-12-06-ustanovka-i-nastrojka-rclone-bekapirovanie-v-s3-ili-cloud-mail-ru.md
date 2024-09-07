---
title: "Установка и настройка rclone. Бэкапирование в s3 или cloud.mail.ru"
date: "2019-12-06"
categories: 
  - Manuals
tags: 
  - "backup"
  - "rclone"
  - "s3"
  - "wordpress"
image:
  path: /commons/1037058_b070_2.jpg
  alt: "Rclone. Бэкапирование"
---

> **Rclone** - это свободное программное обеспечение для синхронизации и копирования файлов между различными хранилищами и устройствами. Оно позволяет создавать и управлять remote-репозиториями на различных типах хранилищ, включая Amazon S3, Google Drive, Microsoft OneDrive, Dropbox и многие другие.
{: .prompt-tip }

В связи с тем, что Яндекс закручивает гайки в своих сервисах, из-за чего наблюдаются перебои с Яндекс Диском, точнее с протоколом `webdav`, который я использовал для хранения бэкапа сайта, пришлось искать другие варианты. Выбор пал на программу `rclone` и хранилище от mail.ru.

## Установка rclone

Скачиваем `rclone` и распаковываем его

```sh
$ curl -O https://downloads.rclone.org/rclone-current-linux-amd64.zip
$ unzip rclone-current-linux-amd64.zip
$ cd rclone-v1.50.2-linux-amd64/
```

Меняем расположение исполняемого файла и настраиваем права

```sh
$ sudo cp rclone /usr/sbin/
$ sudo chown root:root /usr/sbin/rclone
$ sudo chmod 755 /usr/sbin/rclone
```

Устанавливаем мануал для `rclone`

```sh
$ sudo mkdir -p /usr/local/share/man/man1
$ sudo cp rclone.1 /usr/local/share/man/man1/
$ sudo mandb
```

На этом установка окончена

## Обновление rclone

Удаляем версию, установленную из архива

```sh
$ sudo rm /usr/sbin/rclone
```

Скачиваем новую версию и устанавлваем ее

```sh
$ wget https://downloads.rclone.org/v1.53.2/rclone-v1.53.2-linux-amd64.rpm
$ sudo yum -y localinstall rclone-v1.53.2-linux-amd64.rpm
```

## Конфигурирование rclone под s3 от mail.ru

Для начала необходимо зарегистрироваться на портале mcs.mail.ru. Создать аккаунт в объектном хранилище, где будет сгенерирован `Access Key ID` и `Secret Key`. Эти данные необходимо сохранить.  
Затем надо создать `Бакет` - логическая сущность, которая помогает организовать хранение объектов (т.е. грубо говоря каталог, где будут лежать наши файлы).

Запускаем команду для создания конфигурации под хранилище

```sh
$ rclone config
n/s/q> n

# Укажите имя подключения к удаленному хранилищу
name> mailru-s3
...
4 / Amazon S3 Compliant Storage Provider
...

# Выберите тип хранилища, указываем s3:
Storage> 4
...
10 / Any other S3 compatible provider
...

# Выберите поставщика услуг, указываем Other:
provider> 10

# Выберите тип ввода учетных данных (вручную или из переменных окружения)
# Для ввода данных вручную укажите false:
env_auth> 1

# Вводим Access Key ID
access_key_id> ...

# Вводим Secret Key
secret_access_key> ...

# Вводим регион, я указал 1
region> 1

# Вводим Endpoint Можно узнать на портале: mcs.mail.ru - объектное хранилище
endpoint> hb.bizmrg.com

# Укажите регион создания бакета
# оставьте поле пустым и нажмите Enter:
location_constraint>

# Установите права доступа ACL, private:
acl> 1

# После этого можно отказаться от расширенной конфигурации:
Edit advanced config? (y/n)
y) Yes
n) No
y/n> n
```

Далее отобразиться конфигурация, которую мы создали, сохраняем ее и выходим

```
y) Yes this is OK
e) Edit this remote
d) Delete this remote
y/e/d> y

e) Edit existing remote
n) New remote
d) Delete remote
r) Rename remote
c) Copy remote
s) Set configuration password
q) Quit config
e/n/d/r/c/s/q> q
```

## Команды для работы с хранилищем

Список команд и поддерживаемых флагов можно найти на официальном сайте приложения `rcolne`

Просмотр списка бакетов:

```sh
$ rclone lsd mailru-s3:
```

Создание нового бакета

```sh
$ rclone mkdir mailru-s3:itdraft-testbucket
```

Просмотр списка файлов в бакете

```sh
$ rclone ls mailru-s3:itdraft-bucket
```

Копирование файлов с локальной машины в хранилище

```sh
$ rclone copy /mnt/storage/itdraft.ru/backup mailru-s3:itdraft-bucket
```

Синхронизация файлов на локальной машине и в хранилище

```sh
$ rclone sync /mnt/storage/itdraft.ru/backup mailru-s3:itdraft-bucket
```

Копирование файлов из хранилища на локальную машину

```sh
$ rclone copy mailru-s3:itdraft-bucket/backup_itdraft_ru.sql.gz /home/test
```

Не синхронизировать файлы младше 1 дня

```sh
$ rclone --min-age 1d --delete-excluded sync /mnt/storage/itdraft.ru/backup mailru-s3:itdraft-bucket
```

Не синхронизировать файлы старше 7 дней

```sh
$ rclone --max-age 7d --delete-excluded sync /mnt/storage/itdraft.ru/backup mailru-s3:itdraft-bucket
```

Удалить файлы младше 7 дней

```sh
$ rclone --max-age 7d delete mailru-s3:itdraft-bucket
```

Удалить файлы старше 7 дней

```sh
$ rclone --min-age 7d delete mailru-s3:itdraft-bucket
```

## Конфигурирование rclone для работы с cloud.mail.ru

Запускаем команду для создания конфигурации под хранилище

```sh
$ rclone config
e/n/d/r/c/s/q> n
name> cloud-mailru
Storage> mailru
user> user@mail.ru
Password
y) Yes type in my own password
g) Generate random password
y/g> y
Enter the password:
password:
Confirm the password:
password:
speedup_enable> 1
Edit advanced config? (y/n)
y) Yes
n) No
y/n> n
```

Далее отобразиться конфигурация, которую мы создали, сохраняем ее и выходим

```
y) Yes this is OK
e) Edit this remote
d) Delete this remote
y/e/d> y
Current remotes:

Name                 Type
====                 ====
cloud-mailru         mailru
mailru-s3            s3

e) Edit existing remote
n) New remote
d) Delete remote
r) Rename remote
c) Copy remote
s) Set configuration password
q) Quit config
e/n/d/r/c/s/q> q
```

## Пример скрипта бэкапирования дампа базы MySQL в хранилище

Логика скрипта бэкапирования следующая:

- локально хранятся бэкапы базы за 8 дней (т.к. они небольшого размера)
- в облаке хранится бэкапы базы за 1 месяц
- в лог пишется информация о количестве локальных бэкапов и какие были удалены

Скрипт через crontab выполняется ежедневно

```bash
#!/bin/sh

# Переменные
SRC=/mnt/storage/itdraft.ru/backup
DEST=cloud-mailru:/itdraft-backup

# Дата-Врем
DATETIME="$(date +%Y%m%d_%H%M%S)"

# Ящики
mail_from=admin@itdraft.ru
mail_to=admin@itdraft.ru
mail_copy=admin@itdraft.ru

# БД
mysqluser=%user%
mysqlpassword=%pass%
mysqlbase=%database%

# Логи
LOG_FILE=$SRC/backup_mysql.log

# Хранение
DAYS=8
DAYSDEST=1M

# Создаем каталог, если его еще нет
mkdir -p $SRC

# Время запуска скрипта в логах
echo "$DATETIME INFO: Start" | tee "$LOG_FILE"

# Создаем дамп базы локально и архивируем
mysqldump -u $mysqluser -p$mysqlpassword $mysqlbase | gzip > $SRC/$DATETIME.backup_itdraft_ru.sql.gz

# Копируем в облако
rclone copy $SRC $DEST --log-file "$LOG_FILE"

# Удаляем в облаке файлы старше 1 месяца
rclone --min-age $DAYSDEST delete $DEST --log-file "$LOG_FILE"

# Информация для лог-файла: Find local MYSQL dump
echo "Local MYSQL dump" | tee -a "$LOG_FILE"
du -csh --time $SRC/*.sql.gz | tee -a "$LOG_FILE"
echo "Deleted" | tee -a "$LOG_FILE"
find $SRC/*.sql.gz -type f -mtime +$DAYS -print | tee -a "$LOG_FILE"

# Удаляем локальные копии, старше 8 дней
find $SRC/*.sql.gz -type f -mtime +$DAYS -exec rm -f {} \;

# Время завершения скрипта в логах
echo "$DATETIME INFO: End" | tee -a "$LOG_FILE"

# Отправляем лог на почту
echo "itdraft.ru: $DATETIME | deleted files" | mail -v -A yandex -s "itdraft.ru: $DATETIME | deleted files" -a "$LOG_FILE" -r $mail_from $mail_to
```

Для бэкапирования самого сайта у меня запускается похожий скрипт раз в неделю, но из-за другой периодичности запуска:

- локально хранится 1 полный бэкап файлов сайта (т.к. этот бэкап весит на несколько порядков больше, чем база)
- в облаке хранятся бэкапы за месяц (т.е. 4 шт.)

## UPD 13.12.2019

Если конфигурировали rclone от одного пользователя, а скрипт бэкапирования запускается от пользователя root, надо скопировать конфигурационный файл rclone

```sh
$ sudo mkdir -p /root/.config/rclone/
$ sudo cp /home/%user%/.config/rclone/rclone.conf /root/.config/rclone/rclone.conf
```

## UPD 27.10.2020

Проблема с клонирование в облако mail.ru разработчики исправили.  
В версии `rclone v1.53.2` клонирование работает корректно.
