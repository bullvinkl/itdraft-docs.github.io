---
title: "Установка и настройка OpenVPN (клиента и сервера) и Easy-RSA 3 в CentOS 7"
date: "2019-04-18"
categories: 
  - Linux
  - OpenVPN
tags: 
  - "centos"
  - "openvpn"
image:
  path: /commons/1387342_082d_2.jpg
  alt: "Установка и настройка OpenVPN"
---

> **OpenVPN** - бесплатное решение для создания виртуальных частных сетей (VPN), позволяющее организовывать зашифрованные каналы типа точка-точка или сервер-клиенты. Программа поддерживает множество операционных систем, включая Windows, MAC, Linux, Android и iOS. 
> **Easy-RSA** — программа для создания и ведения инфраструктуры открытых ключей (PKI) в openVPN

## Установка необходимого софта

Добавляем репозиторий EPEL и обновляемся

```sh
$ sudo yum install epel-release -y
$ sudo yum update -y
```

Устанавливает `OpenVPN 2.4` и `Easy-RSA 3`

```sh
$ sudo yum install openvpn easy-rsa -y
```

Проверим их версии

```sh
$ sudo openvpn --version
$ sudo ls -lah /usr/share/easy-rsa/
```

![Проверка версий OpenVPN и Easy-RSA](/assets/img/posts/2019/04/18/wp_openvpn_1.png){: w="300" }
_Проверка версий OpenVPN и Easy-RSA_


## Настройка Easy-RSA 3

Скопируем скрипты `easy-rsa` в каталог `/etc/openvpn/`

```sh
$ sudo cp -r /usr/share/easy-rsa /etc/openvpn/
```

Переходим в каталог `/etc/openvpn/easy-rsa/3/` и создаем там файл `vars`

```sh
$ sudo cd /etc/openvpn/easy-rsa/3/
$ sudo nano vars
set_var EASYRSA                 "$PWD"
set_var EASYRSA_PKI             "$EASYRSA/pki"
set_var EASYRSA_DN              "cn_only"
set_var EASYRSA_REQ_COUNTRY     "RU"
set_var EASYRSA_REQ_PROVINCE    "Moscow"
set_var EASYRSA_REQ_CITY        "Moscow"
set_var EASYRSA_REQ_ORG         "My Organisation"
set_var EASYRSA_REQ_EMAIL       "admin@itdraft.ru"
set_var EASYRSA_REQ_OU          "IT department"
set_var EASYRSA_KEY_SIZE        4096
set_var EASYRSA_ALGO            rsa
set_var EASYRSA_CA_EXPIRE       7500
set_var EASYRSA_CERT_EXPIRE     3650
set_var EASYRSA_NS_SUPPORT      "no"
set_var EASYRSA_NS_COMMENT      "CERTIFICATE AUTHORITY"
set_var EASYRSA_EXT_DIR         "$EASYRSA/x509-types"
set_var EASYRSA_SSL_CONF        "$EASYRSA/openssl-1.0.cnf"
set_var EASYRSA_DIGEST          "sha512"
```

Делаем файл исполняемым

```sh
$ sudo chmod +x vars
```

## Создание ключа и сертификата для OpenVPN Сервера

Прежде чем создавать ключ, нам нужно инициализировать каталог `PKI` и создать ключ `CA`.

```sh
$ cd /etc/openvpn/easy-rsa/3/
$ sudo ./easyrsa init-pki
Note: using Easy-RSA configuration from: ./vars

init-pki complete; you may now create a CA or requests.
Your newly created PKI dir is: /etc/openvpn/easy-rsa/3/pki

$ sudo ./easyrsa build-ca
```

На этом необходимо придумать пароль для своего CA-ключа, чтобы сгенерировались файлы `ca.crt` и `ca.key` в каталоге `pki` 
Этот пароль нам потребуется дальше

![Создаем корневой сертификат](/assets/img/posts/2019/04/18/wp_openvpn_2.png){: w="300" }
_Создаем корневой сертификат_

Создадим ключ сервера (название сервера `srv-openvpn`)

```sh
$ sudo ./easyrsa gen-req srv-openvpn nopass
```

- опция `nopass` - отключение пароля для `srv-openvpn`

![Создаем ключ сервера](/assets/img/posts/2019/04/18/wp_openvpn_3.png){: w="300" }
_Создаем ключ сервера_

Подпишем ключ `srv-openvpn` используя наш CA-сертификат

```sh
$ sudo ./easyrsa sign-req server srv-openvpn
```

В процессе у нас спросят пароль, который мы задавали ранее

![Подписываем ключ, используя CA-сертификат](/assets/img/posts/2019/04/18/wp_openvpn_4.png){: w="300" }
_Подписываем ключ, используя CA-сертификат_

Проверим файлы сертификата, что бы убедится, что сертификаты сгенерировались без ошибок

```sh
$ sudo openssl verify -CAfile pki/ca.crt pki/issued/srv-openvpn.crt
pki/issued/srv-openvpn.crt: OK
```

Все сертификата OpenVPN сервера созданы.

- Корневой сертификат расположен: `pki/ca.crt`
- Закрытый ключ сервера расположен: `pki/private/srv-openvpn.key`
- Сертификат сервера расположен: `pki/issued/srv-openvpn.crt`

### Создание ключа клиента

Сгенерируем ключ клиента `client-01`

```sh
$ sudo ./easyrsa gen-req client-01 nopass
```

![Генерируем ключ клиента](/assets/img/posts/2019/04/18/wp_openvpn_5.png){: w="300" }
_Генерируем ключ клиента_

Теперь подпишем ключ `client-01`, используя наш CA сертификат

```sh
$ sudo ./easyrsa sign-req client client-01
```

В процессе у нас спросят пароль, который мы задавали ранее

![Подписываем ключ клиента, используя корневой сертификат](/assets/img/posts/2019/04/18/wp_openvpn_6.png){: w="300" }
_Подписываем ключ клиента, используя корневой сертификат_

Проверим файлы сертификата

```sh
$ sudo openssl verify -CAfile pki/ca.crt pki/issued/client-01.crt
pki/issued/client-01.crt: OK
```

## Дополнительная настройка OpenVPN сервера

Сгенерируем ключ Диффи-Хеллмана

```sh
$ sudo ./easyrsa gen-dh
```

![Генерация ключа Диффи-Хеллмана](/assets/img/posts/2019/04/18/wp_openvpn_7.png){: w="300" }
_Генерация ключа Диффи-Хеллмана_


Если мы в дальнейшем планируем отзывать клиентские сертификаты, нам необходимо сгенерировать `CRL ключ`

```sh
$ sudo ./easyrsa gen-crl
```

В процессе у нас спросят пароль, который мы задавали ранее

![Генерируем CRL-ключ](/assets/img/posts/2019/04/18/wp_openvpn_8.png){: w="300" }
_Генерируем CRL-ключ_

Для того, что бы отозвать сертификат надо выполнить команду:

```sh
$ sudo ./easyrsa revoke client-02
```

где
- `client-02` имя сертификата, который мы отзываем

Все необходимые сертификаты созданы, теперь их надо скопировать в директории

Копируем сертификаты сервера

```sh
$ sudo cp pki/ca.crt /etc/openvpn/server/
$ sudo cp pki/issued/srv-openvpn.crt /etc/openvpn/server/
$ sudo cp pki/private/srv-openvpn.key /etc/openvpn/server/
```

Копируем сертификаты клиента

```sh
$ sudo cp pki/ca.crt /etc/openvpn/client/
$ sudo cp pki/issued/client-01.crt /etc/openvpn/client/
$ sudo cp pki/private/client-01.key /etc/openvpn/client/
```

Копируем ключи `DH` и `CRL`

```sh
$ sudo cp pki/dh.pem /etc/openvpn/server/
$ sudo cp pki/crl.pem /etc/openvpn/server/
```

> Проверить, надо ли перегенерировать `CRL` и заново копировать его в каталог `/etc/openvpn/server/` после отзыва сертификата
{: .prompt-warning }

## Настройка OpenVPN сервера

Создадим файл конфигурации `server.conf`

```sh
$ cd /etc/openvpn/
$ sudo nano server.conf
# OpenVPN Port, Protocol and the Tun
port 1194
proto udp
dev tun

# OpenVPN Server Certificate - CA, server key and certificate
ca /etc/openvpn/server/ca.crt
cert /etc/openvpn/server/srv-openvpn.crt
key /etc/openvpn/server/srv-openvpn.key

# DH and CRL key
dh /etc/openvpn/server/dh.pem
crl-verify /etc/openvpn/server/crl.pem

# Network Configuration - Internal network
# Redirect all Connection through OpenVPN Server
server 10.10.1.0 255.255.255.0
push "redirect-gateway def1"

# Using the DNS from https://dns.watch
push "dhcp-option DNS 84.200.69.80"
push "dhcp-option DNS 84.200.70.40"

# Enable multiple client to connect with same Certificate key
duplicate-cn

# TLS Security
cipher AES-256-CBC
tls-version-min 1.2
tls-cipher TLS-DHE-RSA-WITH-AES-256-GCM-SHA384:TLS-DHE-RSA-WITH-AES-256-CBC-SHA256:TLS-DHE-RSA-WITH-AES-128-GCM-SHA256:TLS-DHE-RSA-WITH-AES-128-CBC-SHA256
auth SHA512
auth-nocache

# Other Configuration
keepalive 20 60
persist-key
persist-tun
comp-lzo yes
daemon
user nobody
group nobody

# OpenVPN Log
log-append /var/log/openvpn.log
verb 3
```

## Настройка Firewalld

Активируем модуль ядра port-forwarding

```sh
$ echo 'net.ipv4.ip_forward = 1' | sudo tee -a /etc/sysctl.conf
$ sudo sysctl -p
net.ipv4.ip_forward = 1
```

Добавим службу `openvpn` в firewall, и интерфейс `tun0` в доверенную зону

```sh
$ sudo firewall-cmd --permanent --add-service=openvpn
$ sudo firewall-cmd --permanent --zone=trusted --add-interface=tun0
```

Активируем `MASQUERADE` для доверенной зоны firewall

```sh
$ sudo firewall-cmd --permanent --zone=trusted --add-masquerade
```

Активируем `NAT`

```sh
$ sudo SERVERIP=$(ip route get 84.200.69.80 | awk 'NR==1 {print $(NF-2)}')
$ sudo firewall-cmd --permanent --direct --passthrough ipv4 -t nat -A POSTROUTING -s  10.10.1.0/24 -o $SERVERIP -j MASQUERADE
```

Перезапустим firewall

```sh
$ sudo firewall-cmd --reload
```

Запустим OpenVPN и добавим его в автозагрузку

```sh
$ sudo systemctl start openvpn@server
$ sudo systemctl enable openvpn@server
```

Проверим

```sh
$ sudo netstat -plntu
$ sudo systemctl status openvpn@server
```

![Проверяем, запущен ли OpenVPN](/assets/img/posts/2019/04/18/wp_openvpn_9-1024x348.png){: w="300" }
_Проверяем, запущен ли OpenVPN_

## Настройка OpenVPN клиента

Создадим файл конфигурации `client-01.ovpn`

```sh
$ cd /etc/openvpn/client
$ sudo nano client-01.ovpn
client
dev tun
proto udp

remote xx.xx.xx.xx 1194

ca ca.crt
cert client-01.crt
key client-01.key

cipher AES-256-CBC
auth SHA512
auth-nocache
tls-version-min 1.2
tls-cipher TLS-DHE-RSA-WITH-AES-256-GCM-SHA384:TLS-DHE-RSA-WITH-AES-256-CBC-SHA256:TLS-DHE-RSA-WITH-AES-128-GCM-SHA256:TLS-DHE-RSA-WITH-AES-128-CBC-SHA256

resolv-retry infinite
compress lzo
nobind
persist-key
persist-tun
mute-replay-warnings
verb 3
```

В строке `remote xx.xx.xx.xx 1194` надо прописать IP-адрес вместо `xx.xx.xx.xx`

Теперь для надо заархивировать сертификаты `ca.crt` и `client-01.crt`, ключ клиента `client-01.key`, файл конфигурации `client-01.ovpn`, и передать их на ПК, который будет подключаться к OpenVPN серверу

Установим архиватор `zip` и создадим архив с файлами

```sh
$ sudo yum install zip unzip -y
$ sudo cd /etc/openvpn/
$ sudo zip client/client-01.zip client/*
```

Пробуем подключиться с другого ПК к OpenVPN серверу и смотрим лог:

```sh
$ sudo tail -f /var/log/openvpn.log
```

![Смотрим log-файл OpenVPN сервера](/assets/img/posts/2019/04/18/wp_openvpn_10-1024x171.png){: w="300" }
_Смотрим log-файл OpenVPN сервера_