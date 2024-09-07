---
title: "Установка почтового сервера iRedMail на CentOS 7. Часть 8. Белые и черные списки. Web-интерфейс"
date: "2019-03-25"
categories: 
  - Linux
  - iRedMail
tags: 
  - "amavisd"
  - "centos"
  - "iredmail"
  - "whitelist"
image:
  path: /commons/virtualisationimage.png
  alt: "Установка iRedMail"
---

> **Белый список** адресов электронной почты содержит одобренные вами адреса электронной почти или доменных имен, с которых можно отправлять письма в ваш домен.  
> **Черный список** адресов электронной почты содержит адреса электронной почти или доменных имен, сообщения с которых не должны попадать в ваш домен.
{: .prompt-tip }

Документация по управлению белым и черным списком можно посмотреть на официальном сайте iRedMail.

Этот функционал доступен в платной версии iRedMail, но в бесплатной версии есть python-скрипт, с помощью которого можно управлять Белыми и черными списками

Например, для добавления доменного имени в белый или черный список надо выполнить команду:

```sh
$ python /opt/iredapd/tools/wblist_admin.py --add --whitelist @example.com
$ python /opt/iredapd/tools/wblist_admin.py --add --blacklist @example.com
```

Для удаления доменного имени из белого или черного списка надо выполнить команду:

```sh
$ python /opt/iredapd/tools/wblist_admin.py --delete --whitelist @example.com
$ python /opt/iredapd/tools/wblist_admin.py --delete --blacklist @example.com
```

Для просмотра белого или черного списка выполним команду:

```sh
$ python /opt/iredapd/tools/wblist_admin.py --list --whitelist
$ python /opt/iredapd/tools/wblist_admin.py --list --blacklist
```

## Интерфейс Web-админки

Проанализировав python-скрипт можно увидеть, что белые и черные списки хранятся в MySQL-базе `amavisd`

Мне не захотелось использовать `phpMyAdmin` для управления белым и черным списком, по-этому набросал свою админку.

[Скачать (github)](https://github.com/bullvinkl/whitelist)

Возможности админки:

- Добавлять в список
- Выбор тип списка (белый / черный)
- Редактировать запись
- Удалять из списка

![](/assets/img/posts/2019/03/25/wp_whitelist_1-1.png){: w="300" }
_Вэб-интерфейс_

![](/assets/img/posts/2019/03/25/wp_whitelist_2-1.png){: w="300" }
_Вэб-интерфейс_

![](/assets/img/posts/2019/03/25/wp_whitelist_3-1.png){: w="300" }
_Вэб-интерфейс_

![](/assets/img/posts/2019/03/25/wp_whitelist_4.png){: w="300" }
_Исходник письма с отправителем из белого списка_

![](/assets/img/posts/2019/03/25/wp_whitelist_5.png){: w="300" }
_Лог сервера с отправителем из черного списка_

Для установки вэб-интерфейса создаем директорию:

```sh
$ sudo mkdir /var/www/html/whitelist
```

Распаковываем в эту директорию файлы из архива, редактируем файлы:

- в файле `server.php` отредактировать строку 3 (заменить `%password%` на свое значение)

Пароль на базу `amavisd` можно найти в письме, которое вам было отправлено после установки mail-сервера iRedMail

Ограничиваем доступ к вэб-интерфейсу управления белыми / черными списками по ip:

```sh
$ sudo nano /etc/nginx/templates/misc.tmpl
...
location ~ ^/whitelist/$ {
    allow %ip%;
    deny all;
}
```

где

- `%ip%` — ip-адрес, которому разрешен доступ

Перезагружаем nginx

```sh
$ sudo systemctl restart nginx
```
