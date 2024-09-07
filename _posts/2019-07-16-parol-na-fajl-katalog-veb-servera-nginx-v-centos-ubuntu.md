---
title: "Пароль на файл/каталог вэб-сервера Nginx в Centos/Ubuntu"
date: "2019-07-16"
categories: 
  - Linux
  - Nginx
tags: 
  - "centos"
  - "nginx"
  - "ubuntu"
image:
  path: /commons/910838_84d6.jpg
  alt: "Пароль на файл/каталог в Nginx"
---

> **.htpasswd** - это файл, используемый для хранения паролей пользователей в системе аутентификации HTTP на веб-сервере Apache. В этом файле хранятся учетные данные пользователей, разделенные на отдельные строки, каждая из которых содержит имя пользователя и пароль, разделенные символом двоеточия (:).
{: .prompt-tip }

Для того, что бы сделать доступ по паролю в каталог вэб-сервера Nginx для начала необходимо сгенерировать файл с логином/паролем `.htpasswd`
Это можно сделать с помощью утилиты от вэб-сервера Apache, либо про помощи php, либо с помощью bash.

Установим утилиту от вэб-сервера Apache:

Для Centos:

```sh
$ sudo yum install httpd-tools
```

Для Ubuntu/Debian:

```sh
$ sudo apt-get install apache2-utils
```

Сгенерируем пароль:

```sh
$ sudo htpasswd -c /var/www/example.ru/public_html/.htpasswd username
```

где:
- `/var/www/example.ru/public_html/` – путь к каталогу
- `username` – имя пользователя, которое мы будем использовать для аутентификации


Сгенерируем пароль для файла `.htpasswd` при помощи `php`:

```sh
$ php -r 'echo crypt("your_password", "salt");'
```

где:
- `your_password` - ваш пароль
- `salt` - соль для пароля, должна содержать минимум 2 символа из набора “0-9 A-Z a-z”

Далее нам надо создать сам файл `.htpasswd` и вписать в него данные в формате:

```
username:password
```

где:
- `username` – имя пользователя, которое мы будем использовать для аутентификации
- `password` - наш сгенерированный пароль


Сгенерируем файл `.htpasswd` при помощи bash:

```sh
$ printf "USER:$(openssl passwd -crypt PASSWORD)\n" | sudo tee -a /var/www/example.ru/public_html/.htpasswd
```

где:
- `USER` – имя пользователя, которое мы будем использовать для аутентификации
- `PASSWORD` - наш пароль
- `/var/www/example.ru/public_html/` - каталог, куда сохранится файл


Я использовал именно этот метод

Редактируем файл конфигурации NGINX, в данном случае нашего виртуального хоста

```sh
$ sudo nano /etc/nginx/sites-available/example.ru.conf
. . .
location /test {
        auth_basic "Password-protected Area";
        auth_basic_user_file /etc/nginx/.htpasswd;
	#autoindex on; # для режима просмотра файлов и директорий
}
```

Проверяем корректность настроек и перезапускаем иэб-сервер

```sh
$ nginx -t
$ sudo service nginx restart
```

Например, что бы закрыть каталог wp-admin — в блоке server указываем

```
location = /wp-login.php {
        
        auth_basic "Restricted";
        auth_basic_user_file /var/www/example.ru/public_html/.htpasswd;
        include fastcgi_params;
        fastcgi_pass unix:/var/run/php-fpm/php-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        }
```
