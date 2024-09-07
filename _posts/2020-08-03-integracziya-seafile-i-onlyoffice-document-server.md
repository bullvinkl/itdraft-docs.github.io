---
title: "Интеграция SeaFile и OnlyOffice Document Server"
date: "2020-08-03"
categories: 
  - Storage-System
tags: 
  - "nginx"
  - "onlyoffice"
  - "seafile"
image:
  path: /commons/1334578_287d_2.jpg
  alt: "SeaFile и OnlyOffice"
---

> **OnlyOffice** —  бесплатный офисный пакет и экосистема приложений для совместной работы. Он состоит из онлайн-редакторов текстовых документов, электронных таблиц, презентаций, форм и PDF-файлов, а также платформы для совместной работы в комнатах.
{: .prompt-tip }

Ранее на сайте рассматривались статьи по установке сервера хранения файлов [SeaFile]({% post_url 2020-02-13-ustanovka-seafile-7-1-0-nginx-percona-na-centos-7 %}) и установке сервера редактирования документов [OnlyOffice Document Server]({% post_url 2020-07-21-ustanovka-onlyoffice-document-server-postgresql-nginx-na-centos-8 %})

Рассмотрим вариант интеграции этих двух сервисов

Останавливаем сервисы `seafile`, `seahub`

```sh
$ sudo systemctl stop seafile seahub
```

Редактируем конфиг Nginx

```sh
$ sudo nano /etc/nginx/sites-available/seafile.conf
# Required for only office document server
map $http_x_forwarded_proto $the_scheme {
        default $http_x_forwarded_proto;
        "" $scheme;
    }

map $http_x_forwarded_host $the_host {
        default $http_x_forwarded_host;
        "" $host;
    }

map $http_upgrade $proxy_connection {
        default upgrade;
        "" close;
    }
[...]
    location /onlyofficeds/ {

        # THIS ONE IS IMPORTANT ! - Trailing slash !
        proxy_pass https://onlyoffice.example.ru/;

        client_max_body_size 100M; # Limit Document size to 100MB
        proxy_read_timeout 3600s;
        proxy_connect_timeout 3600s;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection $proxy_connection;
        proxy_http_version 1.1;

        # THIS ONE IS IMPORTANT ! - Subfolder and NO trailing slash !
        proxy_set_header X-Forwarded-Host $the_host/onlyofficeds;

        proxy_set_header X-Forwarded-Proto $the_scheme;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
```

Проверяем

```sh
$ sudo nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

Перезапускаем Nginx

```sh
$ sudo systemctl restart nginx
```

Редактируем конфиг `seahub_settings.py`

```sh
$ sudo nano /opt/seafile/conf/seahub_settings.py
[...]
# Enable Only Office
ENABLE_ONLYOFFICE = True
VERIFY_ONLYOFFICE_CERTIFICATE = True
ONLYOFFICE_APIJS_URL = 'https://seafile.example.ru/onlyofficeds/web-apps/apps/api/documents/api.js'
ONLYOFFICE_FILE_EXTENSION = ('doc', 'docx', 'ppt', 'pptx', 'xls', 'xlsx', 'odt', 'fodt', 'odp', 'fodp', 'ods', 'fods')
ONLYOFFICE_EDIT_FILE_EXTENSION = ('docx', 'pptx', 'xlsx')
```

Запускаем ранее остановленные сервисы

```sh
$ sudo systemctl start seafile seahub
```

> Важно. При интеграции Seafile и Onlyoffice в тестовой среде (в виртуальных машинах VirtualBox) не работало редактирование в onlyoffice: вэб-интерфейс редактора документов загружался, но далее появлялась ошибка.  
> При установке интеграции серверов в боевой среде, и использовании реальных доменных имен, все заработало штатно.
{: .prompt-warning }
