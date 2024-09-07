---
title: "Обновление ownCloud вручную"
date: "2020-07-16"
categories: 
  - Storage-System
tags: 
  - "centos"
  - "debian"
  - "owncloud"
  - "ubuntu"
  - "apache"
image:
  path: /commons/1162000_fab6_2.jpg
  alt: "Обновление ownCloud"
---

> **ownCloud** — это свободное и открытое веб-приложение для синхронизации данных, общего доступа к файлам.
{: .prompt-tip }

Переключаемся на пользователя `root`. Все дальнейшие действия будут выполняться от этого пользователя

```sh
$ sudo - su
```

Бэкапируем директорию, в которую установлен ownCloud

```sh
# rsync -avpP /var/www/owncloud /opt/backups/
```

Делаем дамп базы данных

```sh
# mysqldump -u root -p owncloud > /opt/backups/owncloud-`date +%F`.sql
Enter password:
```

Включаем режим обслуживания с помощью утилиты `occ` (расположена в каталоге, куда установлен ownCloud)

```sh
# cd /var/www/owncloud
# sudo -u www-data php occ maintenance:mode --on
```

Останавливаем вэб-сервер Apache

```sh
# systemctl stop apache2
```

Скачиваем релиз ownCloud в каталог `/tmp` (На момент написания статьи финальная стабильная версия owncloud-10.4.1)

```sh
# wget https://download.owncloud.org/community/owncloud-10.4.1.tar.bz2 -P /tmp/
```

Подготавливаемся к обновлению: переименовываем каталог с установленным ownCloud, распаковываем скаченный архив

```sh
# cd
# mv /var/www/owncloud /var/www/owncloud-bak
# tar xjf /tmp/owncloud-10.4.1.tar.bz2 -C /var/www/
```

Назначаем права (вэб-сервер Apache работает от пользователя `www-data`)

```sh
# chown -R www-data:www-data /var/www/owncloud
```

Копируем каталог с данными из старого ownCloud (в конфигурационном файле указано расположение данных, возможно у вас он вынесен на отдельный диск, тогда этот пункт можно пропустить)

```sh
# rsync -avpP /var/www/owncloud-bak/data /var/www/owncloud/
```

Заменяем дефолтный конфиг на рабочий (из предыдущей версии ownCloud)

```sh
# rsync -avpP /var/www/owncloud-bak/config /var/www/owncloud/
```

У меня в конфиге была указана директория `apps-external`, при распаковке архива ее не было, создаем эту директорию

```sh
# mkdir /var/www/owncloud/apps-external
# chown www-data:www-data /var/www/owncloud/apps-external
```

Обновляем ownCloud

```sh
# cd /var/www/owncloud
# sudo -u www-data php /var/www/owncloud/occ upgrade
```

Во время обновления возникла ошибка

> Repair warning: You have incompatible or missing apps enabled that could not be found or updated via the marketplace.  
> Repair warning: Please install or update the following apps manually or disable them with: occ app:disable files_videoplayer  
> ...  
> OC\RepairException: Upgrade is not possible  
> Update failed
{: .prompt-warning }

Система ругается на `files_videoplayer`, отключим его

```sh
# sudo -u www-data php /var/www/owncloud/occ app:disable files_videoplayer
```

Обновляемся

```sh
# sudo -u www-data php /var/www/owncloud/occ upgrade
```

Проверяем установленную версию ownCloud

```sh
# sudo -u www-data php /var/www/owncloud/occ -V
```

Выключаем режим обслуживания

```sh
# sudo -u www-data php /var/www/owncloud/occ maintenance:mode --off
```

Перезапускаем вэб-сервер Apache

```sh
# systemctl restart apache2
```
