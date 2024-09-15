---
title: "Установка почтового сервера iRedMail на CentOS 7. Часть 3. Алиасы, вэб-интерфейс для работы с алиасами"
date: "2019-02-08"
categories: 
  - Communication-System
tags: 
  - "alias"
  - "centos"
  - "iredmail"
image:
  path: /commons/analytics_ai_x-ray.png
  alt: "Установка iRedMail"
---

> **Алиас** - короткое, удобное для запоминания имя, использующееся вместо более длинного и сложного имени; наиболее часто используется в приложениях электронной почты.
{: .prompt-tip }

## Включаем возможность отправлять письма через алиас

Редактируем конфиг Postfix `/etc/postfix/main.cf`, удаляем строчку:

```
reject_sender_login_mismatch
```

в версии `iRedMail 0.9.9` этой строки уже не было

Перезагружаем postfix

```sh
$ sudo systemctl restart postfix
```

Редактируем конфиг iRedAPD `/opt/iredapd/settings.py`, добавляем строку:

```
reject_sender_login_mismatch
```

в версии `iRedMail 0.9.9` эта строка уже была добавлена

Перезагружаем iRedAPD

```sh
$ sudo systemctl restart iredapd
```

## Установка phpMyAdmin и настройка NGINX

Устанавливаем phpMyAdmin:

```sh
$ sudo yum install phpmyadmin
```

Делаем линк

```sh
$ sudo ln -s /usr/share/phpMyAdmin /var/www/html/pma
```

Ограничиваем доступ к phpMyAdmin по ip

```sh
$ sudo nano /etc/nginx/templates/misc.tmpl
...
location ~ ^/pma/$ {
    allow %ip%;
    deny all;
}
```

где
- `%ip%` - ip-адрес, которому разрешен доступ к phpMyAdmin

Перезагружаем Nginx

```sh
$ sudo systemctl restart nginx
```

## WEB-интерфейс для управления алиасами

Мне не захотелось устанавливать громоздкий `postfixadmin` для возможности управлением алиасами, по-этому быстренько набросал свою админку

Из мануала iRedMail, алиасы добавляются SQL-запросом

```sh
INSERT INTO alias (address, domain, active) VALUES ('alias@mydomain.com', 'mydomain.com', 1);
INSERT INTO forwardings (address, forwarding, domain, dest_domain, is_list, active) VALUES ('alias@mydomain.com', 'someone@test.com', 'mydomain.com', 'test.com', 1, 1);
```

Возможности админки:

- Добавлять алиас
- Редактировать алиас
- Удалять алиас

В дальнейшем добавлю возможность активировать/деактивировать активность алиаса

[Скачать (github)](https://github.com/bullvinkl/alias)

Для установки вэб-интерфейса создаем директорию:

```sh
$ sudo mkdir /var/www/html/alias
```

Распаковываем в эту директорию файлы из архива, редактируем файлы:  
- в файле `index.php` - отредактировать строки 225, 226  
- в файле `server.php` - отредактировать строку 3 (прописать пароль к базе между пустых кавычек)

Где находится пароль от базы Mysql для пользователя `vmailadmin`: После установки почтового сервера на почтовый ящик postmaster@domain.ru падает письмо со всеми паролями. Либо пароль можно найти в конфигах

Ограничиваем доступ к вэб-интерфейсу управлением алиасами по ip

```sh
$ sudo nano /etc/nginx/templates/misc.tmpl
...
location ~ ^/alias/$ {
    allow %ip%;
    deny all;
}
```

где
- `%ip%` - ip-адрес, которому разрешен доступ к phpMyAdmin

Перезагружаем Nginx

```bash
$ sudo systemctl restart nginx
```

## UPD 27.03.2019

Обновил вэб-админку, добавил следующие возможности:

- Делать алиас активным / не активным при добавлении записи
- Делать алиас активным / не активным при редактировании записи
- При редактировании записи сделал активным поле "алиас" и добавил проверку, есть ли редактируемый алиас в базе
