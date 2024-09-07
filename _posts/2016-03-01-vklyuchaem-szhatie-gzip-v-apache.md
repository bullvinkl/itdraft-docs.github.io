---
title: "Включаем сжатие gzip в Apache"
date: "2016-03-01"
categories: 
  - Web
tags: 
  - "apache"
  - "centos"
  - "gzip"
image:
  path: /commons/computer.jpg
  alt: "Включаем сжатие gzip в Apache"
---

> **OpenVPN** - это свободная реализация технологии Виртуальной Частной Сети (VPN) с открытым исходным кодом. Она позволяет создавать зашифрованные каналы типа точка-точка или сервер-клиенты между компьютерами, обеспечивая безопасное и защищенное соединение.
{: .prompt-tip }

Проверяем подключен ли модуль в файле `/etc/httpd/conf/httpd.conf` для этого открываем файл и ищем строчку

```sh
$ cat /etc/httpd/conf/httpd.conf | grep mod_deflate.so
LoadModule deflate_module modules/mod_deflate.so
```

Добавляем в конфигурацию следующие строчки

```sh
$ sudo nano /etc/httpd/conf/httpd.conf

# mod_deflate configuration
<IfModule mod_deflate.c>

# Restrict compression to these MIME types
AddOutputFilterByType DEFLATE text/plain
AddOutputFilterByType DEFLATE text/html
AddOutputFilterByType DEFLATE application/xhtml+xml
AddOutputFilterByType DEFLATE text/xml
AddOutputFilterByType DEFLATE application/xml
AddOutputFilterByType DEFLATE application/x-javascript
AddOutputFilterByType DEFLATE text/javascript
AddOutputFilterByType DEFLATE text/css

# Level of compression (Highest 9 - Lowest 1)
DeflateCompressionLevel 9

# Netscape 4.x has some problems.
BrowserMatch ^Mozilla/4 gzip-only-text/html

# Netscape 4.06-4.08 have some more problems
BrowserMatch ^Mozilla/4\.0[678] no-gzip

# MSIE masquerades as Netscape, but it is fine
BrowserMatch \bMSI[E] !no-gzip !gzip-only-text/html

</IfModule>
```

Перезагружаем Apache

```sh
$ sudo service httpd restart
```

Модуль `mod_deflate` позволяет экономить до 70% траффика на страницах с HTML-содержимым. В зависимости от количества графики и других несжимаемых элементов на ваших сайтах, экономия может составлять около 10% от всего траффика.
