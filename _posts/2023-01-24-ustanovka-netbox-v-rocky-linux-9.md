---
title: "Установка NetBox в Rocky Linux 9"
date: "2023-01-24"
categories: 
  - Asset-Management
tags: 
  - assetmanager
  - cmdb
  - itsm
  - netbox
  - rocky-linux
  - postgresql
  - redis
  - python
  - nginx
  - firewall
image:
  path: /commons/nn_guide-min.png
  alt: "Установка NetBox в Rocky Linux 9"
---

> **Netbox** — веб приложение с открытым исходным кодом, разработанное для управления и документирования компьютерных сетей. Изначально Netbox придуман командой сетевых инженеров DigitalOcean специально для системных администраторов.
{: .prompt-tip }

## Установка PostgreSQL 15 из репозитория

Добавляем репозиторий PostgreSQL
```sh
$ sudo dnf -y install https://download.postgresql.org/pub/repos/yum/reporpms/EL-9-x86_64/pgdg-redhat-repo-latest.noarch.rpm
$ sudo dnf -y update
```

Отключаем модуль `postgresql`, что бы PostgreSQL не устанавливался из дефолтных репозиториев
```sh
$ sudo dnf -qy module disable postgresql
```

Устанавливаем PostgreSQL 15
```sh
$ sudo dnf -y install postgresql15-server
```

Инициализируем БД
```sh
$ sudo /usr/pgsql-15/bin/postgresql-15-setup initdb
```

Правим конфиг `pg_hba.conf`
```sh
$ sudo nano /var/lib/pgsql/15/data/pg_hba.conf
...
host    all             all             127.0.0.1/32            md5          
# IPv6 local connections:
host    all             all             ::1/128                 md5
...
```

Запускаем PostgreSQL
```sh
$ sudo systemctl enable postgresql-15
$ sudo systemctl start postgresql-15
$ sudo systemctl status postgresql-15
```

Устанавливаем пароль пользователя `postgres`
```sh
$ sudo -u postgres psql
ALTER USER postgres WITH PASSWORD 'mysuperpass';
```

Создаем пользователя `netbox` и базу
```sh
=# CREATE USER netbox WITH ENCRYPTED PASSWORD 'passwdnetbox';
=# CREATE DATABASE netboxdb OWNER netbox;
=# GRANT ALL PRIVILEGES ON DATABASE netboxdb TO netbox;
=# \q
```

## Установка Redis

> **Redis** - быстрое хранилище данных типа «ключ‑значение» в памяти, с открытым исходным кодом

Устанавливаем Redis из дефолтного репозитория
```sh
$ sudo dnf -y install redis
```

Задаем пароль
```sh
$ sudo nano /etc/redis/redis.conf
...
# requirepass foobared
requirepass myredissuperpasswd
```

Запускаем Redis
```sh
$ sudo systemctl start redis
$ sudo systemctl enable redis
$ sudo systemctl status redis
```

## Установка Netbox

Устанавливаем необходимые пакеты
```sh
$ sudo dnf -y install gcc libxml2-devel libxslt-devel libffi-devel libpq-devel openssl-devel redhat-rpm-config git
```

Создаем пользователя `netbox`
```sh
$ sudo useradd -r -d /opt/netbox -s /usr/sbin/nologin netbox
```

Создаем каталог, переходим в него, клонируем репозиторий
```sh
$ sudo mkdir -p /opt/netbox
$ cd /opt/netbox
$ sudo git clone -b master --depth 1 https://github.com/netbox-community/netbox.git .
```

Меняем владельца каталога
```sh
$ sudo chown -R netbox:netbox /opt/netbox
$ cd /opt/netbox/netbox/netbox
```

Копируем и переименовываем конфиг
```sh
$ sudo -u netbox cp configuration_example.py configuration.py
```

Генерим ключ
```sh
$ sudo -u netbox python3 ../generate_secret_key.py
#oi*fQIE&jNqFjaJd8l@j^%=*_Kh%P*I8MyjIIVCD^zcd4vHH_
```

Правим конфиг `configuration.py`
```sh
$ sudo -u netbox nano configuration.py
...
ALLOWED_HOSTS = ['*']

DATABASE = {
    'NAME': 'netboxdb',         # Database name
    'USER': 'netbox',               # PostgreSQL username
    'PASSWORD': 'passwdnetbox',           # PostgreSQL password
    'HOST': 'localhost',      # Database server
    'PORT': '',               # Database port (leave blank for default)
    'CONN_MAX_AGE': 300,      # Max database connection age
}

REDIS = {
    'tasks': {
        'HOST': 'localhost',
        'PORT': 6379,
        # Comment out `HOST` and `PORT` lines and uncomment the following if using Redis Sentinel
        # 'SENTINELS': [('mysentinel.redis.example.com', 6379)],
        # 'SENTINEL_SERVICE': 'netbox',
        'USERNAME': '',
        'PASSWORD': 'myredissuperpasswd',
        'DATABASE': 0,
        'SSL': False,
        # Set this to True to skip TLS certificate verification
        # This can expose the connection to attacks, be careful
        # 'INSECURE_SKIP_TLS_VERIFY': False,
    },
    'caching': {
        'HOST': 'localhost',
        'PORT': 6379,
        # Comment out `HOST` and `PORT` lines and uncomment the following if using Redis Sentinel
        # 'SENTINELS': [('mysentinel.redis.example.com', 6379)],
        # 'SENTINEL_SERVICE': 'netbox',
        'USERNAME': '',
        'PASSWORD': 'myredissuperpasswd',
        'DATABASE': 1,
        'SSL': False,
        # Set this to True to skip TLS certificate verification
        # This can expose the connection to attacks, be careful
        # 'INSECURE_SKIP_TLS_VERIFY': False,
    }
}
SECRET_KEY = '#oi*fQIE&jNqFjaJd8l@j^%=*_Kh%P*I8MyjIIVCD^zcd4vHH_'
...
```

Запускаем скрипт установки Netbox
```sh
$ sudo -u netbox /opt/netbox/upgrade.sh
```

Переключаемся на пользователя `root`
```sh
$ sudo su
```

Активируем виртуальную среду Python и переходим в каталог `netbox`
```sh
# source /opt/netbox/venv/bin/activate
(venv) # /opt/netbox/netbox
```

Создаем админа Nebox
```sh
(venv) # python3 manage.py createsuperuser
Username (leave blank to use 'root'): admin
Email address: admin@itdraft.ru
Password: admin
Password (again): admin
Superuser created successfully.
```

Создаем сим линк скрипта `netbox-housekeeping.sh` в ежедневные задания `cron`. Сценарий `netbox-housekeeping.sh` используется для очистки среды Netbox, он удаляет просроченные задачи, старые сеансы или записи с истекшим сроком действия.
```sh
(venv) # sudo ln -s /opt/netbox/contrib/netbox-housekeeping.sh /etc/cron.daily/netbox-housekeeping
```

Копируем конфиг `gunicorn.py`
```sh
(venv) # sudo -u netbox cp /opt/netbox/contrib/gunicorn.py /opt/netbox/gunicorn.py
```

Копируем файлы Systemd Unit
```sh
(venv) # sudo cp -v /opt/netbox/contrib/*.service /etc/systemd/system/
'/opt/netbox/contrib/netbox-rq.service' -> '/etc/systemd/system/netbox-rq.service'
'/opt/netbox/contrib/netbox.service' -> '/etc/systemd/system/netbox.service'
```

Выполняем `daemon-reload`, что бы Systemd нашел новые сервисы
```sh
(venv) # sudo systemctl daemon-reload
```

Запускаем сервисы и смотрим статус
```sh
(venv) # sudo systemctl start netbox netbox-rq
(venv) # sudo systemctl enable netbox netbox-rq

(venv) # sudo systemctl status netbox
(venv) # sudo systemctl status netbox-rq
```

Выходим из виртуальной среды Python и переключаемся с пользователя `root`
```sh
(venv) # deactivate
# exit
```

## Установка Nginx

Добавляем репозиторий Nginx
```sh
$ sudo nano /etc/yum.repos.d/nginx.repo
[nginx-stable]
name=nginx stable repo
baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=1
enabled=1
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true

[nginx-mainline]
name=nginx mainline repo
baseurl=http://nginx.org/packages/mainline/centos/$releasever/$basearch/
gpgcheck=1
enabled=0
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true
```

Устанавливаем Nginx
```sh
$ sudo dnf -y install nginx
```

Отключаем дефолтный конфиг
```sh
$ sudo mv /etc/nginx/conf.d/default.conf /etc/nginx/conf.d/default.conf.disable
```

Копируем и редактируем конфиг из дистрибутива Netbox
```sh
$ sudo cp /opt/netbox/contrib/nginx.conf /etc/nginx/conf.d/netbox.conf
$ sudo nano /etc/nginx/conf.d/netbox.conf
server {
    listen [::]:443 ssl ipv6only=off;

    # CHANGE THIS TO YOUR SERVER'S NAME
    server_name _;

    ssl_certificate /etc/nginx/ssl/server.crt;
    ssl_certificate_key /etc/nginx/ssl/server.key;

    client_max_body_size 25m;

    location /static/ {
        alias /opt/netbox/netbox/static/;
    }

    location / {
        proxy_pass http://127.0.0.1:8001;
        proxy_set_header X-Forwarded-Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

server {
    # Redirect HTTP traffic to HTTPS
    listen [::]:80 ipv6only=off;
    server_name _;
    return 301 https://$host$request_uri;
}
```

Создаем каталог для SSL-сертификата
```sh
$ sudo mkdir /etc/nginx/ssl/
```

Копируем наш сертификат
```sh
$ sudo mv /home/user/server.* /etc/nginx/ssl/
```

Отключаем SELinux
```sh
$ sudo setenforce 0
$ sudo grubby --update-kernel ALL --args selinux=0
```

Запускаем Nginx, и добавляем сервис в автозагрузку
```sh
$ sudo systemctl enable --now nginx
```

## Настройка Firewall

Открываем `http` и `https` порты
```sh
$ sudo firewall-cmd --add-servic={http,https} --permanent
$ sudo firewall-cmd --reload
```

## Установка плагина для NetBox

Для примера установим плагин `netbox-dns`

Переключаемся на пользователя `root`

```sh
$ sudo su
# cd ~
```

Активируем виртуальную среду Python
```sh
# source /opt/netbox/venv/bin/activate
```

Устанавливаем плагин из репозитория Python
```sh
(venv) # pip install netbox-dns
```

Добавляем строку в файл `local_requirements.txt`
```sh
(venv) # nano /opt/netbox/local_requirements.txt
...
netbox-dns
```

Редактируем конфиг `configuration.py`, добавляем в него строку
```sh
(venv) # nano /opt/netbox/netbox/netbox/configuration.py
...
PLUGINS = [
    "netbox_dns"
]
```

Запускаем процесс миграции (добавляются необходимые таблицы, устанавливаются недостающие модули Python)
```sh
(venv) # cd /opt/netbox/netbox/
(venv) # python3 manage.py migrate
```

Перезапускаем сервисы, смотрим их статус
```sh
(venv) # systemctl restart netbox netbox-rq
(venv) # systemctl status netbox netbox-rq
```

Выходим из виртуальной среды Python
```sh
(venv) # deactivate
```

## UPD 19.12.2023 (Ошибка при установка netbox, связанная с Redis)

При установки Netbox столкнулся с ошибкой, связанной с паролем в Redis.  
Убираем его, т.к. по дефолту коннект к Redis разрешен только локально

```sh
$ sudo nano /etc/redis/redis.conf
...
# requirepass foobared
# requirepass myredissuperpasswd

$ sudo -u netbox nano configuration.py
...
'PASSWORD': '',
...
```
