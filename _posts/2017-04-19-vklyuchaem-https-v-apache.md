---
title: "Включаем HTTPS в Apache"
date: "2017-04-19"
categories: 
  - Web
tags: 
  - "apache"
  - "centos"
  - "https"
  - "mod_ssl"
image:
  path: /commons/computer.jpg
  alt: "Включаем HTTPS в Apache"
---

> **Apache** - это свободное программное обеспечение с открытым исходным кодом, предназначенное для создания веб-сервера. Это кроссплатформенное ПО, которое позволяет устанавливать и настраивать веб-сервер на различных типах серверов.
{: .prompt-tip }

Чтобы подключить SSL шифрование нам надо установить `OpenSSL` и `mod-ssl` (расширение для Apache)

```sh
$ sudo yum install mod_ssl openssl
```

Генерируем собственный сертификат используя OpenSSL, для этого

- Генерируем приватный ключ с 2048-битным шифрованием
- Генерируем запроса на сертификат CSR
- Генерируем самоподписанный ключ на 356 дней

```sh
$ sudo openssl genrsa -out ca.key 2048
$ sudo openssl req -new -key ca.key -out ca.csr
$ sudo openssl x509 -req -days 365 -in ca.csr -signkey ca.key -out ca.crt
```

Перемещаем полученные файлы в правильное место

```sh
$ sudo mv ca.crt /etc/pki/tls/certs
$ sudo mv ca.key /etc/pki/tls/private/ca.key
$ sudo mv ca.csr /etc/pki/tls/private/ca.csr
```

Обновим конфигурационный файл Apache SSL

```sh
$ sudo vi +/SSLCertificateFile /etc/httpd/conf.d/ssl.conf

SSLCertificateFile /etc/pki/tls/certs/ca.crt
SSLCertificateKeyFile /etc/pki/tls/private/ca.key
```

После чего надо перезапустить Apache

```sh
$ sudo service httpd restart
```

## Настройка виртуальных хостов

Все аналогично тому, как вы создавали `VirtualHosts` для `HTTP` на `80` порту - все тоже для `HTTPS` на порту `443`. Типичный виртуальный хост для 80 порта выглядит так

```
<VirtualHost *:80>
        <Directory /var/www/vhosts/yoursite.com/httpdocs>
        AllowOverride All
        </Directory>
        DocumentRoot /var/www/vhosts/yoursite.com/httpdocs
        ServerName yoursite.com
</VirtualHost>
```

Чтобы включить `ssl`, необходимо добавить следующее в верхней части вашего файла

```
NameVirtualHost *:443
<VirtualHost *:443>
        SSLEngine on
        SSLCertificateFile /etc/pki/tls/certs/ca.crt
        SSLCertificateKeyFile /etc/pki/tls/private/ca.key
        <Directory /var/www/vhosts/yoursite.com/httpsdocs>
        AllowOverride All
        </Directory>
        DocumentRoot /var/www/vhosts/yoursite.com/httpsdocs
        ServerName yoursite.com
</VirtualHost>
```

И перезапустить Apache

```sh
$ sudo service httpd restart
```

## Настройка брандмауэра

Нам надо открыть порт `443`, чтоб можно было подключаться к сайту по `https`

```sh
$ sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT
$ sudo /sbin/service iptables save
$ sudo iptables -L -v
```
