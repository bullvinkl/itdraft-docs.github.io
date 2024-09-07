---
layout: post
title: "Установка 1С:Сервера взаимодействия + Nginx + Minio"
date: "2023-12-08"
categories:
  - Storage-System
  - Web
  - Communication-System
tags:
  - 1c
  - centos
  - debian
  - java
  - linux
  - minio
  - nginx
  - postgresql
  - rocky-linux
image:
  path: /commons/p9lahdjjxckbx3hsm3uqaioj4bu.png
  alt: "Установка 1С:Сервера взаимодействия + Nginx + Minio"
---

> Система взаимодействия – это механизм внутри «1С», позволяющий организовать взаимодействие сотрудников прямо в базе. 1С: Сервер взаимодействия — это программное обеспечение реализующее серверную часть системы взаимодействия. В сервер взаимодействия входят следующие компоненты:  
> - Сервер системы взаимодействия
> - Распределенное хранилище Hazelcast
> - Поисковый кластер Elasticsearch
{: .prompt-tip }

## Установка СУБД PostgreSQL

Добавим официальный репозиторий и устанавливаем PostgreSQL
```sh
$ sudo apt -y install gnupg2
$ sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
$ wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
$ sudo apt update
$ sudo apt -y install postgresql-15
$ sudo ln -s /usr/lib/postgresql/15/bin/* /usr/sbin/
```

Создаем пользователя, базу, подключаем расширение.

Подключимся к консоли postgres
```sh
$ sudo -u postgres psql
```

Зададим пароль пользователю `postgres`
```sh
=# \password postgres
Enter new password for user "postgres": passwd
Enter it again:
```

Создадим пользователя
```sh
=# CREATE USER db_user WITH PASSWORD 'userpass';
```

Создадим базу `cs_db`
```sh
=# CREATE DATABASE cs_db WITH OWNER = db_user;
```

Подключаемся к базе и добавим расширение необходимое для сервера взаимодействия
```sh
=# \c cs_db
=# CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
=# \q
```

## Установка Java 11

Устанавливаем софт
```sh
$ sudo apt install -y default-jdk
$ sudo apt install -y default-jre
```

```sh
$ sudo nano /etc/profile.d/java.sh
export PATH=$PATH:/usr/lib/jvm/java-11-openjdk-amd64/bin/
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64/

$ source /etc/profile.d/java.sh
```

## Установка 1С:Сервер взаимодействия

Закачиваем дистрибутив на сервер, в домашнюю директорию пользователя, распаковываем архив, устанавливаем ПО
```sh
$ tar -vxf 1c_cs_24.0.29_linux_x86_64.tar.gz 
$ sudo ./1ce-installer-cli install --ignore-signature-warnings
```

Настраиваем утилиту `ring`
```sh
$ find / -name ring
/opt/1C/1CE/components/1c-enterprise-ring-0.19.5+12-x86_64/ring
/opt/1C/1CE/components/1c-cs-hazelcast-3.9.4-24-x86_64/lib/ring
/opt/1C/1CE/components/1c-cs-elasticsearch-5.6.12-23-x86_64/lib/ring
/opt/1C/1CE/components/1c-cs-server-small-24.0.29-x86_64/lib/ring
$ sudo nano /etc/profile.d/ring.sh
export PATH=$PATH:/opt/1C/1CE/components/1c-enterprise-ring-0.19.5+12-x86_64
```

Команда `ring` работает корректно исключительно под пользователем root. Переключаемся на него, продолжаем установку
```sh
$ sudo su
# source /etc/profile.d/java.sh
# source /etc/profile.d/ring.sh
```

Инициализация сервера взаимодействия 1С
```sh
# useradd cs_user
# mkdir -p /var/cs/cs_instance
# chown cs_user:cs_user /var/cs/cs_instance
# ring cs instance create --dir /var/cs/cs_instance --owner cs_user
# ring cs --instance cs_instance service create --username cs_user --java-home $JAVA_HOME --stopped
```

Инициализация сервера Hazelcast
```sh
# useradd hc_user
# mkdir -p /var/cs/hc_instance
# chown hc_user:hc_user /var/cs/hc_instance
# ring hazelcast instance create --dir /var/cs/hc_instance --owner hc_user
# ring hazelcast --instance hc_instance service create --username hc_user --java-home $JAVA_HOME --stopped
```

Инициализация сервера Elasticsearch
```sh
# useradd elastic_user
# mkdir -p /var/cs/elastic_instance
# chown elastic_user:elastic_user /var/cs/elastic_instance
# ring elasticsearch instance create --dir /var/cs/elastic_instance --owner elastic_user
# ring elasticsearch --instance elastic_instance service create --username elastic_user --java-home $JAVA_HOME --stopped
```

Настройка параметров JDBC драйвера PostgreSQL
```sh
# ring cs --instance cs_instance jdbc pools --name common set-params --url jdbc:postgresql://localhost:5432/cs_db?currentSchema=public
# ring cs --instance cs_instance jdbc pools --name common set-params --username db_user
# ring cs --instance cs_instance jdbc pools --name common set-params --password userpass
# ring cs --instance cs_instance jdbc pools --name privileged set-params --url jdbc:postgresql://localhost:5432/cs_db?currentSchema=public
# ring cs --instance cs_instance jdbc pools --name privileged set-params --username db_user
# ring cs --instance cs_instance jdbc pools --name privileged set-params --password userpass
```

Настройка WebSocket
```sh
# ring cs --instance cs_instance websocket set-params --hostname 192.168.12.25
# ring cs --instance cs_instance websocket set-params --port 8181
```

По очереди включим сервисы сервера взаимодействия 1С
```sh
# ring elasticsearch --instance elastic_instance service start
# ring hazelcast --instance hc_instance service start
# ring cs --instance cs_instance service start
```

Смотрим статусы
```sh
# ring cs --instance cs_instance service status
# ring hazelcast --instance hc_instance service status
# ring elasticsearch --instance elastic_instance service status
```

Проверка
```sh
# curl http://localhost:8087/rs/health
{"status":"UP","mainDbOk":true,"allShardsOk":true,"hazelcast":{"available":true,"members":["127.0.0.1:5701"]},"elasticsearchOk":true,"mediaClusterOk":false,"mediaServers":{},"pushOk":false}
```

Инициализация базы данных сервера
```sh
# curl -Sf -X POST -H "Content-Type: application/json" -d "{ \"url\" : \"jdbc:postgresql://localhost:5432/cs_db\", \"username\" : \"db_user\", \"password\" : \"userpass\", \"enabled\" : true }" -u admin:admin http://localhost:8087/admin/bucket_server
```

Пример успешного ответа
```sh
{"id":"242e3b6c-2934-460c-9197-f070e769eee8","url":"jdbc:postgresql://localhost:5432/cs_db","username":"db_user","password":"userpass","lastUsedAt":null,"enabled":true,"deleted":false}
```

## Установка web-сервера Nginx

Выходим из пользователя root и добавляем ключ репозитория
```sh
# exit
$ wget --quiet -O - https://nginx.org/keys/nginx_signing.key | sudo apt-key add -
```

Добавляем репозиторий Nginx
```sh
$ sudo nano /etc/apt/sources.list.d/nginx.list
# NGINX repo
deb https://nginx.org/packages/mainline/debian/ bullseye nginx
deb-src https://nginx.org/packages/mainline/debian bullseye nginx
```

Устанавливаем Nginx
```sh
$ sudo apt update
$ sudo apt install -y nginx
```

## Настройка 1С:Сервера взаимодействия для работы по https

В данном примере я использую готовый ssl-сертификат. Но можно установить и настроить утилиту certbot для получения бесплатного ssl-сертифииката от Let's Encrypt. В этом случае данный пункт придется выполнять при перевыпуске сертификата, т.е. раз в 3 месяца

Переключаемся на полmзователя root и создаем каталог
```sh
$ sudo su
# mkdir -p /var/cs/cs_instance/data/security
```

Импортируем сертификат и ключ в хранилище JKS. Для этого объединим их в единый файл PKCS12
```sh
# openssl pkcs12 -export -in server.crt -inkey server.key -out /var/cs/cs_instance/data/security/pkcs.p12 -name wildcard
Enter Export Password: pkcspass
Verifying - Enter Export Password: pkcspass
```

Сгенерируем хранилище ключей JKS с импортированным файлом pkcs.p12
```sh
# keytool -importkeystore -destkeystore /var/cs/cs_instance/data/security/websocket-keystore.jks -srckeystore /var/cs/cs_instance/data/security/pkcs.p12 -srcstoretype PKCS12 -alias wildcard
Enter destination keystore password: jkspass
Re-enter new password: jkspass
Enter source keystore password: pkcspass
```

Включим режим защищённого подключения wss
```sh
# ring cs --instance cs_instance websocket set-params --wss true
# ring cs --instance cs_instance websocket set-params --keystore-path /var/cs/cs_instance/data/security/websocket-keystore.jks
# ring cs --instance cs_instance websocket set-params --keystore-password jkspass
# ring cs --instance cs_instance websocket set-params --keystore-format JKS

# ring cs --instance cs_instance websocket list-params
```

## Настройка https в Nginx

Создаем каталог для сертификатов
```sh
# mkdir -p /etc/nginx/ssl/
```

Сохраняем готовый сертификат и ключ в него
```sh
# mv server.* /etc/nginx/ssl/
```

Генерируем dhparam.pem (процедура идет достаточно долго). dhparam - это простое число, используемое в алгоритме Диффи-Хеллмана для обмена сессионными ключами с клиентом
```sh
# openssl dhparam -out /etc/nginx/ssl/dhparam.pem 4096
```

Создаем конфиг Nginx
```sh
# nano /etc/nginx/conf.d/1c-interaction.conf
server {
    listen 80;
    listen 443 ssl;
    server_name 1c-interaction.itdraft.ru;
    charset utf-8;

    ssl_certificate /etc/nginx/ssl/server.crt;
    ssl_certificate_key /etc/nginx/ssl/server.key;
    ssl_session_timeout 5m;
    ssl_session_cache shared:SSL:5m;
    ssl_dhparam /etc/nginx/ssl/dhparam.pem;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers 'ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES256-SHA384:ECDHE-RSA-AES128-SHA256:ECDHE-RSA-AES256-SHA:ECDHE-ECDSA-AES256-SHA:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES128-SHA:DHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES256-SHA256:DHE-RSA-AES256-SHA:DHE-RSA-CAMELLIA256-SHA:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-RSA-SEED-SHA:DHE-RSA-CAMELLIA128-SHA:HIGH:!aNULL:!eNULL:!LOW:!3DES:!MD5:!EXP:!PSK:!SRP:!DSS';
    ssl_prefer_server_ciphers on;

    if ($scheme != "https") {
        return 301 https://$host$request_uri;
    }
    server_tokens off;
	
    root /opt/1C/1CE/components/1c-cs-site-0.2.11-x86_64;
    index index.html index.htm;

    access_log /var/log/nginx/1c-interaction_access.log;
    error_log /var/log/nginx/1c-interaction_error.log;
}
```

Проверяем конфиг Nginx на валидность и перезапускаем web-сервер
```sh
# nginx -t
# nginx -s reload
или
# systemctl restart nginx
```

Отредактируем файл config.json
```sh
# nano /opt/1C/1CE/components/1c-cs-site-0.2.11-x86_64/config.json
{
        "serverURL": "wss://1c-interaction.itdraft.ru:8181",
        "VAPIDPublicKey": "BEYY_t98CbgoqySRFVyTl36Fq2mV44nhw0g27TZMVNyEZAToazA87Hy9s9PK-9XeYN1DrAh8msQuDp_5YvJ5mPM"
}
```

Параметр VAPIDPublicKey уже был в файле

Вносим изменения в параметр `public-url` сервера взаимодействия
```sh
# ring cs --instance cs_instance site set-params --public-url https://1c-interaction.itdraft.ru
```

Вносим изменения в настройки WebSocket
```sh
# ring cs --instance cs_instance websocket set-params --hostname 1c-interaction.itdraft.ru
```

Проверяем
```sh
# ring cs --instance cs_instance websocket list-params
{hostname='1c-interaction.itdraft.ru', port=8181, keystorePath='/var/cs/cs_instance/data/security/websocket-keystore.jks', keystoreFormat='JKS', keystorePassword='jkspass', wss=true, maxHttpContentLength=128 KB, maxFramePayloadLength=128 KB, pingTimeout=60000, pingInterval=25000, bossThreads=0, workerThreads=0}
```

По очереди выключаем/включим сервисы сервера взаимодействия 1С
```sh
# ring cs --instance cs_instance service stop
# ring hazelcast --instance hc_instance service stop
# ring elasticsearch --instance elastic_instance service stop

# ring elasticsearch --instance elastic_instance service start
# ring hazelcast --instance hc_instance service start
# ring cs --instance cs_instance service start

# ring cs --instance cs_instance websocket list-params
```

## Установка собственного хранилища Minio

Летом 2023 года в Minio были внесены изменения в подсистему безопасности.  
Эти изменения пока не учтены в текущих версия "1С:Сервер взаимодействия".  
Для обхода проблемы следует использовать дистрибутивы Minio, выпущенные до лета 2023 года.

Переключаемся на нашего пользователя. Скачиваем дистрибутив Minio, делаем его исполняемым,
```sh
# exit
$ wget https://dl.min.io/server/minio/release/linux-amd64/archive/minio.RELEASE.2023-05-04T21-44-30Z
$ sudo chmod +x minio.RELEASE.2023-05-04T21-44-30Z
$ sudo mv minio.RELEASE.2023-05-04T21-44-30Z /usr/local/bin/minio
```

Добавляем системного пользователя, меняем владельца
```sh
$ sudo useradd -r minio_user -s /sbin/nologin
$ sudo chown minio_user:minio_user /usr/local/bin/minio
```

Создадим каталог, в котором Minio будет хранить файлы, меняем владельца каталога
```sh
$ sudo mkdir /opt/minio
$ sudo chown minio_user:minio_user /opt/minio
```

Создаем каталог для конфигурационных файлов, ssl-сертфиката для Minio
```sh
$ sudo mkdir /etc/minio
$ sudo chown minio_user:minio_user /etc/minio
```

Создаем конфиг Minio
```sh
$ sudo nano /etc/default/minio
MINIO_ACCESS_KEY=minioadmin
MINIO_SECRET_KEY=minioadmin
MINIO_ROOT_USER=minioadmin
MINIO_ROOT_PASSWORD=minioadmin
MINIO_VOLUMES="/opt/minio/"
MINIO_OPTS="-C /etc/minio --address :9000 --console-address :9001"
MINIO_API_ROOT_ACCESS=on
MINIO_SERVER_URL="https://1c-interaction.itdraft.ru:9000"
```

Скачиваем файл для создания sustemd unid для Minio, редактируем его
```sh
$ curl -O https://raw.githubusercontent.com/minio/minio-service/master/linux-systemd/minio.service
$ sudo nano minio.service
...
User=minio_user
Group=minio_user
...
```

Переместим файл minio.service в каталог /etc/system/system
```sh
$ sudo mv minio.service /etc/systemd/system
```

Перезагрузим все юниты systemd
```sh
$ sudo systemctl daemon-reload
```

Запускаем Minio, добавляем сервис в автозагрузку и проверяем статус
```sh
$ sudo systemctl enable --now minio
$ sudo systemctl status minio
```

## Настройка Minio для работы по https и его подключение к серверу взаимодействия 1С

Копируем наш ssl-сертификат и ключ
```sh
$ sudo cp /etc/nginx/ssl/server.crt /etc/minio/certs/public.crt
$ sudo cp /etc/nginx/ssl/server.key /etc/minio/certs/private.key
```

Меняем влалельца
```sh
$ sudo chown minio_user:minio_user /etc/minio/certs/public.crt
$ sudo chown minio_user:minio_user /etc/minio/certs/private.key
```

Перезапускаем сервис
```sh
$ sudo systemctl restart minio
```

Открываем адрес нашего Minio в браузере
```
https://1c-interaction.itdraft.ru:9001
```

Создаем контейнер cs-bucket
```
Bucket > Create Bucket > Bucket Name: cs-bucket
Access Policy: public
```

Добавляем параметры подключения к хранилищу Minio в 1С:Сервис взаимодействия
```sh
$ curl -Sf -X POST -H 'Content-Type: application/json' -d '{ "apiType": "AMAZON", "storageType": "DEFAULT", "baseUrl": "https://1c-interaction.itdraft.ru:9000", "containerUrl": "https://1c-interaction.itdraft.ru:9000/${container_name}", "containerName": "cs-bucket", "region": "eu-west-1", "accessKeyId": "minioadmin", "secretKey": "minioadmin", "signatureVersion": "V2", "uploadLimit": 1073741824, "downloadLimit": 1073741824, "fileSizeLimit": 104857600, "bytesToKeep": 104857600, "daysToKeep": 31, "pathStyleAccessEnabled": true }' -u admin:admin http://localhost:8087/admin/storage_server
```

## UPD 30.07.2024 Обновление ssl сертификатов

- Обновить сертификат для nginx (/etc/nginx/ssl/)
- Обновить сертификат для minio (/etc/minio/certs/)
- Обновить сертификат для сервера взаимодействия

Сделаем резервную копию и объединим сертифиат и ключ в единый файл PKCS12
```sh
# cd /var/cs/cs_instance/data/security/
# mv pkcs.p12 pkcs.p12.2024
# mv websocket-keystore.jks websocket-keystore.jks.2024
# cd /etc/nginx/ssl/
# openssl pkcs12 -export -in server.crt -inkey server.key -out /var/cs/cs_instance/data/security/pkcs.p12 -name wildcardgge
Enter Export Password: pkcspass
Verifying - Enter Export Password: pkcspass
```

Сгенерируем хранилище ключей JKS с импортированным файлом pkcs.p12
```sh
# keytool -importkeystore -destkeystore /var/cs/cs_instance/data/security/websocket-keystore.jks -srckeystore /var/cs/cs_instance/data/security/pkcs.p12 -srcstoretype PKCS12 -alias wildcardgge
Enter destination keystore password: jkspass
Re-enter new password: jkspass
Enter source keystore password: pkcspass
```

Перезапускаем сервис cs\_instance
```sh
# source /etc/profile.d/java.sh
# source /etc/profile.d/ring.sh
# ring cs --instance cs_instance service stop
# ring cs --instance cs_instance service start
# ring cs --instance cs_instance service status
```

