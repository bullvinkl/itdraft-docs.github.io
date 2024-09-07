---
title: "Локальный APT (Debian / Ubuntu) репозиторий на Centos 7"
date: "2019-12-16"
categories: 
  - Linux
tags: 
  - "debian"
  - "nginx"
  - "repository"
  - "ubuntu"
image:
  path: /commons/1387342_082d_2.jpg
  alt: "Локальный APT репозиторий"
---

> **Репозиторий** — место, где хранятся и поддерживаются какие-либо данные. Чаще всего данные в репозитории хранятся в виде файлов, доступных для дальнейшего распространения по сети.  
> Среди дистрибутивов Linux популярны репозитории с форматом метаданных YUM для дистрибутивов на базе RPM-пакетов, и репозитории с метаданными APT для дистрибутивов на основе DEB-пакетов.
{: .prompt-tip }

Добавляем репозиторий EPEL и устанавливаем софт

```sh
$ sudo yum -y install epel-release
$ sudo yum -y install debmirror
```

Создаем каталог, где будет находиться репозиторий

```sh
$ mkdir -p /var/www/repo/debian
```

Запускаем синхронизацию и зеркалом Яндекса

```sh
$ debmirror --progress --i18n --method=rsync --host=mirror.yandex.ru --dist=buster --ignore-small-errors --ignore-release-gpg --exclude="games" --nosource --arch=amd64 /var/www/repo/debian
```

## Автоматическое обновление репозиториев

Создаем скрипты для автоматического обновления локального зеркала с источником

```sh
$ nano /home/debmirror-debian.sh
#!/bin/sh
arch=amd64
section=main,contrib,non-free
release=buster,buster-updates
server=mirror.yandex.ru
inPath=debian
proto=rsync
outPath=/var/www/repo/debian/
debmirror --arch $arch \
--passive \
--progress \
--nosource \
--i18n \
--ignore-small-errors \
--ignore-release-gpg \
--exclude='games' \
--exclude='/Translation-.*\.bz2' \
--include='/Translation-en.*\.bz2' \
--include='/Translation-ru.*\.bz2' \
--section $section \
--host $server \
--dist $release \
--method $proto \
--root $inPath \
$outPath
```

```sh
$ nano /home/debmirror-debian-security.sh
#!/bin/sh
arch=amd64
section=main,contrib,non-free
release=buster/updates
server=mirror.yandex.ru
inPath=debian-security
proto=rsync
outPath=/var/www/repo/debian-security/
debmirror --arch $arch \
--passive \
--progress \
--nosource \
--i18n \
--ignore-small-errors \
--ignore-release-gpg \
--exclude='games' \
--exclude='/Translation-.*\.bz2' \
--include='/Translation-en.*\.bz2' \
--include='/Translation-ru.*\.bz2' \
--section $section \
--host $server \
--dist $release \
--method $proto \
--root $inPath \
$outPath
```

Используемые параметры:

- `nosourse` — не закачивать пакеты с исходным кодом (экономит место на диске);
- `passive` — пассивный режим загрузки;
- `i18n` — загружать перевод для описания пакетов;
- `host` — URL репозитория;
- `root` — путь к репозиторию на сайте (подкаталог, содержащий репозиторий);
- `method` — метод скачивания (http, ftp, hftp, rsync);
- `progress` — отображение состояния скачивания;
- `dist` — версия дистрибутива ОС (lenny, squeeze, …);
- `arch` — архитектура ОС (i386, amd64);
- `section` — подраздел репозитория с пакетами (main,contrib,non-free; для Ubuntu — main,multiverse,univrese, …);
- `cleanup` — принудительная очистка каталогов репозитория от неизвестного содержимого (неизвестных пакетов);
- `ignore-release-gpg` — игнорировать отсутствие цифровой подписи (полезно при скачивании с некоторых интернет-зеркал официального репозитория, хотя и менее безопасно).

Делаем скрипты исполняемыми

```sh
$ sudo chmod +x /home/debmirror-debian.sh
$ sudo chmod +x /home/debmirror-debian-security.sh
```

Приписываем эти скрипты в crontab для автоматической синхронизации по расписанию

```sh
$ crontab -e
# ежедневно в два часа ночи
0 2 * * * /home/debmirror-debian.sh
# ежедневно в три часа ночи
0 3 * * * /home/debmirror-debian-security.sh
```

Для нормально работы с цифровой подписью надо установить GnuPG и импортировать подпись репозитория

Репозиторий на диске занимает приблизительно 150 Gb

Далее надо поднять вэб-сервер.  
Пример конфигурации Nginx:

```sh
$ sudo cat /etc/nginx/site-avaliable/repo.conf
server {
    listen 80 default_server;
    server_name _;
    root /var/www/repo;
    charset UTF-8;
    default_type text/plain;

    location / {
        autoindex_exact_size off;
        autoindex on;
    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root /usr/share/nginx/html;
    }
}
```

## Некоторые ошибки, которые встречаются в процессе обновления локального репозитория с источником

При первом запуске, будут ошибки:

```
...
[GNUPG:] NO_PUBKEY 04EE7237B7D453EC
...
[GNUPG:] NO_PUBKEY 648ACFD622F3D138
...
[GNUPG:] NO_PUBKEY DCC9EFBF77E11517
```

```sh
$ gpg --no-default-keyring --keyring ~/.gnupg/trustedkeys.gpg --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 04EE7237B7D453EC
$ gpg --no-default-keyring --keyring ~/.gnupg/trustedkeys.gpg --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 648ACFD622F3D138
$ gpg --no-default-keyring --keyring ~/.gnupg/trustedkeys.gpg --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys DCC9EFBF77E11517
```

Посмотреть список gpg-ключей

```sh
$ gpg --list-keys
$ gpg --no-default-keyring --keyring ~/.gnupg/trustedkeys.gpg --list-key
```

Еще некоторые ошибки:

```
Errors:
 Download of dists/sid/Release failed
Failed to download some Release or Release.gpg files!
```

Решение, редактируем файл /etc/debmirror.conf, комментируем строки

```
@dists="sid";
@sections="main,main/debian-installer,contrib,non-free";
@arches="i386";
```

Еще одна ошибка

```
firewall blocking port 11371
```

Решение

```sh
$ gpg --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys E084DAB9
$ gpg -a --export E084DAB9 | sudo apt-key add -
```

## Добавить локальные репозитории на клиентские ПК / сервера

Добавляем локальный репозиторий на клиенте

```sh
$ sudo nano /etc/apt/suorces.list
deb http://192.168.1.21/debian buster main contrib non-free
deb http://192.168.1.21/debian buster-updates main contrib non-free
deb http://192.168.1.21/debian-security buster/updates main contrib non-free
```
