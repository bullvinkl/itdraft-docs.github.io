---
layout: post
title: "Полезные команды (пополняется)"
date: "2024-09-08"
pin: true
categories:
  - Manuals
tags:
  - bash
  - hotkey
image:
  path: /commons/e9c1804a-9ae6-421a-a3a7-f720b8200ab9.webp
  alt: "Полезные Linux команды (пополняется)"
---

> **Полезные команды Linux** - утилиты командной строки, предназначенные для выполнения различных задач в системе Linux. Они позволяют управлять файловой системой, процессами, пользователями, а также выполнять другие важные операции.
{: .prompt-tip }

## SSH

- `ssh-keygen -t rsa -C "rsa-key"`  - генерация ключа, алгоритм `rsa`
- `ssh-keygen -t ed25519 -C "ed25519-key"`  - генерация ключа, алгоритм `ed25519` (`~/.ssh/`  - в каталог сохранятся ключи)
- `ssh-keygen -f ./mykey -t ed25519 -C "ed25519-key"`  - генерация ключа, алгоритм `ed25519`, сохранение в текущем каталоге с именем `mykey` и `mykey.pub`
- `ssh-copy-id username@192.168.1.1`  - копировать публичную часть ключа на удаленный сервер

## TAR GZ — Создание / распаковка архива

- `tar -cvf file.tar /full/path`  - создать `.tar`
- `tar -czvf file.tar.gz /full/path`  - создать `.tar.gz`
- `tar -cjvf file.tar.bz2 /full/path`  - создать `.tar.bz2`
- `tar -xvf file.tar.gz`  - распаковать `.tar.gz`

## Консольные утилиты

- `whoami`  - показывает имя пользователя
- `last -10`   - показывает последние 10 подключений
- `find`  - показать содержимое текущего каталога
- `find /home`  - показать содержимое каталога `/home`
- `nl file.txt`  - нумерация строк файла
- `cat -n`  - отлично заменяет `nl`, более того, `cat -n` умеет и `grep`
- `tree`  - структура в древовидном виде
- `which ping`  - путь до утилиты `ping`
- `whereis ping`  - путь до утилиты `ping`
- `screen ./long-unix-script.sh`  - после запуска команды нажмите `Ctrl + A` и затем `d`, чтобы отключить процесс от текущей сессии. Он будет продолжать работать в фоновом режиме. Для того, чтобы снова подключить процесс, введите:  `screen -r 4980.pts-0.localhost`. Замечание: последняя часть команды здесь - это `id` скрина, который можно узнать с помощью команды `screen -ls`
- `cd -`  - если вы случайно сменили директорию, можно просто вернуться в последнюю
- `sudo !!`  - если вы набрали команду без `sudo`, а потом оказалось, что она необходима. `!!` вообще полезная штука. Если вдруг забыл набрать `cd` перед путем до директории, можно сделать такой же трюк, как и с `sudo`, т.е. `cd !!`
- `mtr`  - мощный инструмент для диагностики сети. Он совмещает в себе функциональность `traceroute` и `ping`
- `pkill [application_name]`  - завершает запущенный процесс. Эта команда особенно полезна, когда приложение не отвечает
- `w`  - показывает, кто на данный момент вошел в систему, наряду с другой полезной информацией такой, как время работы или нагрузкой процессора
- `less`  - читать файл, можно пользоваться `w` и `z` для пролистывания страниц, `/` для поиска текста и т.д.
- `rev`  - переворачивает строчку задом на перед, например `echo abc | rev`
- `tailf`  - синоним `tail -f`
- `users | nl`  - имена всех авторизованных пользователей
- `du -hd 0`  - измеряет размер директории, в которой находитесь
- `du -hd 1`  - измеряет размер всех директорий в директории, в которой находитесь
- `cp /path/to/file.txt{,.backup}`  - результат: `/path/to/file.txt` и `/path/to/file.txt.backup`
- `mv /path/to/file.{txt,log}`  - результат: `/path/to/file.log`
- `>/var/log/daemon.log` - очистить файл
- `:>/var/log/daemon.log` - очистить файл
- `grep -vE "^#|^$" /etc/apt-cacher-ng/acng.conf` - просмотр конфигов без закомментированных строк
- `ss -nltup | column -t`  - `column -t` выстраивает данные в удобочитаемые таблицы. Еще пример: `mount | column -t`
- `echo 1 > /sys/block/sda/device/rescan` - перечитать размер диска `/dev/sda`, выполняется от пользователя `root`
- `growpart /dev/sda 3` - увеличить 3-тью область
- `openssl pkcs12 -export -in certca.pem -inkey privateky.key -out output.pfx`  - создать `PFX`-сертификат
- `curl https://itdraft.ru -H 'User-Agent: GPTBot' -I` - меняем User-Agent в CURL запросе
- `nohup ./script.sh > /dev/null &`  - запуск программы или скрипт а в фоне
- `ps -aux | grep  script` или `pgrep -a script`  - узнать id фоновой программы или скрипта, что бы потом завершить ее (`kill 21536`)

Записать строки в файл, включая спецсимволы, не открывая текстовый редактор:

```sh
sudo tee /etc/apt/apt.conf.d/00proxy <<'EOF' > /dev/null 2>&1
Acquire {
  HTTP::proxy "http://mirror.itdraft.ru:3142";
}
EOF
```

- `echo "proxy=http://mirror.itdraft.ru:3142" | sudo tee /etc/yum.conf > /dev/null 2>&1` - перезаписать `/etc/yum.conf`

- `echo "proxy=http://mirror.itdraft.ru:3142" | sudo tee -a /etc/yum.conf > /dev/null 2>&1` - добавить строку в `/etc/yum.conf`

- `> /dev/null 2>&1` - не выводить результат

## Сочетания клавиш в редакторе Nano

- `CTRL + S`  - сохранить
- `ALT + T`  - очистить все дальше каретки
- `CTRL + C`  - показать номер строки
- `CTRL + K`  - удалить строку
- `CTRL + W`  - поиск

## Сочетание клавиш в консоли

- `CTRL + R`  - поиск по истории
- `CRTL + S`  - остановить вывод логов, например при выводе командой `tail -f /var/log/nginx/access.log`
- `CTRL + Q`  - продолжить вывод логов, например при выводе командой `tail -f /var/log/nginx/access.log`
- `ESC + .` - последний аргумент команды
- `CTRL + L`  - очистить терминал
- `CRTL + R`  - поиск в истории команд терминала
- `CTRL + C`  - прервать работу команды
- `CTRL + Z`  - перевести в фоновую. Чтобы посмотреть задачи, которые сейчас работают в фоне используйте команду `jobs`, а для возврата задачи в нормальный режим - команду `fg`
- `CTRL + P` и `CTRL + N`  - альтернативы клавишам стрелки `вверх` и `вниз`
- `CTRL + A` и `CTRL + E`  - аналоги клавиш `Home` и `End`
- `Ctrl + U`  - удалить весь текст от начала строки до позиции курсора
- `Ctrl + K`  - удалить весь текст от позиции курсора и до конца строки
- `CTRL + W`  - стереть слово перед курсором. Если курсор находится в середине слова, то будут стёрты все символы от курсора до начала слова
- `Alt + Shift + 3`  - ставит в начало строки знак комментария и нажимает `Enter`

## Firewall-cmd

```sh
$ sudo firewall-cmd --list-all
$ sudo firewall-cmd --permanent --zone=public --add-service=https
$ sudo firewall-cmd --permanent --zone=public --add-service={http,https}
$ sudo firewall-cmd --permanent --zone=public --add-port=80/tcp
$ sudo firewall-cmd --permanent --zone=public --add-port=4990-4999/udp
$ sudo firewall-cmd --permanent --zone=public --remove-service=https
$ sudo firewall-cmd --permanent --zone=public --remove-port=80/tcp
$ sudo firewall-cmd --reload
```

## SSL сертификат

- `openssl req -x509 -newkey rsa:2048 -keyout key.pem -out cert.pem -sha256 -days 365`  - самоподписанный сертификат, interactive

- `openssl req -x509 -newkey rsa:2048 -keyout key.pem -out cert.pem -sha256 -days 3650 -nodes -subj "/C=RU/ST=Moscow/L=Moscow/O=Company/OU=IT/CN=CommonNameOrHostname"`  - самоподписанный сертификат, non-interactive

- `openssl req -x509 -newkey rsa:4096 -sha256 -days 3650 -nodes -keyout example.com.key -out example.com.crt -subj "/CN=example.com" -addext "subjectAltName=DNS:example.com,DNS:*.example.com,IP:10.0.0.1"`  - самоподписанный сертификат, `altname`

## Nginx

 Редирект `http > https`

```sh
server {
    listen 80;
    listen 443 default ssl;
    server_name _;
...
    ssl_certificate /etc/nginx/ssl/server.crt;
    ssl_certificate_key /etc/nginx/ssl/server.key;
...
    if ($scheme != "https") {
      return 301 https://$host$request_uri;
    }
...
```

Балансировка

```sh
upstream backend {
    server 192.168.1.1:443;
    server 192.168.1.2:442;
}

server {
...
    location / {
        proxy_pass https://backend;
        ...
    }
}
```

## Docker

Прописать зеркало и разрешить подключение по `http` (необходимо перезапустить сервис Docker)

```sh
$ cat /etc/docker/daemon.json
{
  "registry-mirrors": ["https://registry.itdraft.ru"]
}

{
  "insecure-registries" : ["registry.itdraft.ru:5000"]
}
```

- `docker exec -it %id% /bin/bash`  - подключиться к контейнеру

## GIT

Инициализируем репозиторий и пушим в Git

```sh
$ cd /opt/nginx
$ git init
$ git config --global user.email "admin@itdraft.ru"
$ git config --global user.name  "Maksim Makarov"
$ git add .
$ git commit -m "ver 1.0"
$ git remote add origin ssh://git@gitlab.itdraft.ru:10022/m.makarov/nginx.git
$ git push origin master
```

Пушим изменения в Git

```sh
$ cd /opt/nginx
$ git add .
$ git commit -m "ver 1.2"
$ git push origin master
```

Подтянуть изменения из Git

```sh
$ cd /opt/nginx
$ git pull origin main
```
