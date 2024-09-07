---
title: "Настройка SSO в Nginx, интеграция с Zabbix в Centos 8 / Rocky Linux"
date: "2021-11-19"
categories: 
  - Linux
  - Zabbix
  - Nginx
tags: 
  - "centos"
  - "ldap"
  - "nginx"
  - "rocky-linux"
  - "spnego"
  - "sso"
  - "zabbix"
coverImage: "mockup-2443050_1280.jpg"
image:
  path: /commons/mockup-2443050_1280.jpg
  alt: "Настройка SSO в Nginx, интеграция с Zabbix"
---

> **Технология единого входа (Single sign-on SSO)** — метод аутентификации, который позволяет пользователю переходить из одной системы в другую, не связанную с первой системой, без повторной аутентификации.
{: .prompt-tip }

## Подготовка

Исходные данные:

- на сервере уже установлен Zabbix + Nginx
- в Zabbix настроена LDAP-авторизация, пользователи из AD

Устанавливаем необходимые пакеты для сборки Nginx из исходников

```sh
$ sudo dnf -y install tar gcc unzip gcc-c++ make pcre-devel zlib-devel curl wget git openssl-devel libxml2-devel libxslt-devel gd-devel perl-ExtUtils-Embed gperftools-devel redhat-rpm-config
```

## Сборка динамического модуля Spnego

Проверяем версию Nginx

```sh
$ nginx -v
nginx version: nginx/1.20.1
```

Скачиваем исходники Nginx (нашей установленной версии) и распаковываем их

```sh
$ cd /tmp
$ wget https://nginx.org/download/nginx-1.20.1.tar.gz
$ tar zxvf nginx-*.tar.gz
```

Переходим в каталог

```sh
$ cd nginx-*/
```

Клонируем репозиторий модуля SPNEGO

```sh
$ git clone https://github.com/stnoonan/spnego-http-auth-nginx-module.git
```

Смотрим, с какими опциями собран установленный NGINX

```sh
$ nginx -V
```

Нам нужно все, что идет после `configure arguments:`

Задаем конфигурацию для сборки Nginx из исходников, в конце дописываем `--add-dynamic-module=spnego-http-auth-nginx-module`, что бы собрать динамический модуль

```sh
./configure --prefix=/etc/nginx --sbin-path=/usr/sbin/nginx --modules-path=/usr/lib64/nginx/modules --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --pid-path=/var/run/nginx.pid --lock-path=/var/run/nginx.lock --http-client-body-temp-path=/var/cache/nginx/client_temp --http-proxy-temp-path=/var/cache/nginx/proxy_temp --http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp --http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp --http-scgi-temp-path=/var/cache/nginx/scgi_temp --user=nginx --group=nginx --with-compat --with-file-aio --with-threads --with-http_addition_module --with-http_auth_request_module --with-http_dav_module --with-http_flv_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_mp4_module --with-http_random_index_module --with-http_realip_module --with-http_secure_link_module --with-http_slice_module --with-http_ssl_module --with-http_stub_status_module --with-http_sub_module --with-http_v2_module --with-mail --with-mail_ssl_module --with-stream --with-stream_realip_module --with-stream_ssl_module --with-stream_ssl_preread_module --with-cc-opt='-O2 -g -pipe -Wall -Werror=format-security -Wp,-D_FORTIFY_SOURCE=2 -Wp,-D_GLIBCXX_ASSERTIONS -fexceptions -fstack-protector-strong -grecord-gcc-switches -specs=/usr/lib/rpm/redhat/redhat-hardened-cc1 -specs=/usr/lib/rpm/redhat/redhat-annobin-cc1 -m64 -mtune=generic -fasynchronous-unwind-tables -fstack-clash-protection -fcf-protection -fPIC' --with-ld-opt='-Wl,-z,relro -Wl,-z,now -pie' --add-dynamic-module=spnego-http-auth-nginx-module
```

Собираем

```sh
$ make modules
```

В каталоге `objs` появится файл `ngx_http_auth_spnego_module.so`

## Подключение модуля Spnego в Nginx

Копируем модуль в соответствующий каталог и задаем права

```sh
$ sudo cp objs/ngx_http_auth_spnego_module.so /usr/lib64/nginx/modules/
$ sudo chmod 644 /usr/lib64/nginx/modules/ngx_http_auth_spnego_module.so
```

Добавляем модуль `ngx_http_auth_spnego_module.so` в конфиг Nginx

```sh
$ sudo nano /etc/nginx/nginx.conf
...
pid        /var/run/nginx.pid;

load_module modules/ngx_http_auth_spnego_module.so;

events {
...
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

## Интеграция с Windows AD

Устанавливаем Kerberos клиент

```sh
$ sudo dnf -y install krb5-workstation
```

Редактируем конфиг

```
[logging]
default = FILE:/var/log/krb5libs.log
kdc = FILE:/var/log/krb5kdc.log
admin_server = FILE:/var/log/kadmind.log

[libdefaults]
default_realm = ITDRAFT.RU
default_keytab_name = /etc/nginx/zabbix_srv.keytab
dns_lookup_kdc = false
dns_lookup_realm = false
forwardable = true
ticket_lifetime = 24h

[realms]
ITDRAFT.RU = {
 kdc = srv-dc-01.itdraft.ru
 kdc = srv-dc-02.itdraft.ru
 default_domain = ITDRAFT.RU
 admin_server = srv-dc-01.itdraft.ru
}

[domain_realm]
.itdraft.ru = ITDRAFT.RU
itdraft.ru = ITDRAFT.RU
```

Добавляем в локальный DNS наш zabbix-server "mon.itdraft.ru"  
Прописываем FQDN в /etc/hosts

## Создание учетной записи и keytab в AD

В AD создаем стандартную учетку, в данном примере `zabbix_srv`

Запускаем powershell, Создаем SPN запись

```powershell
> setspn -A HTTP/mon.itdraft.ru@ITDRAFT.RU zabbix_srv
```

Создаем keytab-файл

```powershell
> ktpass /princ HTTP/mon.itdraft.ru@ITDRAFT.RU /mapuser zabbix_srv@ITDRAFT.RU  /crypto ALL /ptype KRB5_NT_PRINCIPAL /out C:\zabbix_srv.keytab /pass *
```

где
- `mon.itdraft.ru `— имя zabbix-сервера;
- `ITDRAFT.RU` — домен;
- `zabbix_srv@ITDRAFT.RU` — учетная запись в AD;
- `pass *` — пароль.

Копируем keytab-файл на наш zabbix-сервер в каталог `/etc/nginx/`

Проверяем на zabbix-сервере

```sh
$ kinit -V -k -t /etc/nginx/zabbix_srv.keytab HTTP/mon.itdraft.ru@ITDRAFT.RU
Using new cache: 1000:47555
Using principal: HTTP/mon.itdraft.ru@ITDRAFT.RU
Using keytab: /etc/nginx/zabbix_srv.keytab
Authenticated to Kerberos v5
```

## Настройка HTTP аутентификации в zabbix

Переходим в раздел: Администрирование - Аутентификация - Настройки HTTP

![](/assets/img/posts/2021/11/19/image.png){: w="300" }

Активируем HTTP аутентификацию

## Настройка SSO в Nginx

Редактируем конфиг Nginx для Zabbix

```sh
$ sudo nano /etc/nginx/conf.d/zabbix.conf
server {
        listen          80;
        listen          443 ssl;
        server_name     mon.itdraft.ru;
		
        auth_gss on;
        auth_gss_realm ITDRAFT.RU;
        auth_gss_keytab /etc/nginx/zabbix_srv.keytab;
        auth_gss_service_name "HTTP/mon.itdraft.ru";
        auth_gss_allow_basic_fallback on;
...
```

Проверяем конфиг Nginx

```sh
$ sudo nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

Перезапускаем Nginx

```sh
$ sudo systemctl restart nginx
```

Теперь необходимо наш сервер добавить в "доверенные", что бы заработало SSO

Для проверки можно запустить Google Chrome с параметрами:

```
"C:\Program Files (x86)\Google\Chrome\Application\chrome.exe" --auth-server-whitelist="mon.itdraft.ru" --auth-negotiate-delegate-whitelist="mon.itdraft.ru"
```
