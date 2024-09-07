---
title: "Установка 1C Server + Postgres PRO + Apache + Эмулятор HASP в Centos 8"
date: "2020-10-08"
categories: 
  - Linux
  - PostgreSQL
  - 1C
tags: 
  - "1c"
  - "apache"
  - "centos"
  - "hasp"
  - "haspemu"
  - "linux"
  - "postgres-pro"
  - "postgresql"
  - "firewall"
image:
  path: /commons/156398bb67a3d4ee7ab97075a1137791.jpg
  alt: "1C Server + Postgres PRO"
---

> 1С Сервер - это виртуальный сервер, предназначенный для использования с программами 1С. Он выступает посредником между сервером баз данных и клиентскими компьютерами, беря на себя тяжелые вычислительные задачи и разгружая клиентские компьютеры.
{: .prompt-tip }

## Подготовка

Обновляемся, добавляем репозиторий EPEL, устанавливаем софт

```sh
$ sudo dnf -y update
$ sudo dnf -y install epel-release
$ sudo dnf -y install wget bzip2 traceroute net-tools nano bind-utils telnet htop atop iftop lsof git rsync policycoreutils-python-utils tar zip unzip
```

Изменим hostname сервера

```sh
$ sudo hostnamectl set-hostname server1c
$ sudo nano /etc/hosts
...
192.168.11.235 server1c
```

На клиентской машине сервер должен отвечать на ping по доменному имени

## Установка Postgres PRO

Добавляем репозиторий Postgres Pro

```sh
$ sudo rpm -i http://repo.postgrespro.ru/pgpro-12/keys/centos.rpm
$ sudo dnf makecache
```

Устанавливаем PostgreSQL PRO std

```sh
$ sudo dnf -y install postgrespro-std-12
```

Проверяем статус

```sh
$ sudo systemctl status postgrespro-std-12
```

Удаляем базу, которая создалась по-умолчанию

```sh
$ sudo rm -rf /var/lib/pgpro/std-12/data
```

Инициализируем БД, модифицируем настройки под работу с 1с и добавляем поддержку русского языка

```sh
$ sudo /opt/pgpro/std-12/bin/pg-setup initdb --tune=1c --locale=ru_RU.UTF-8
```

> без --locale=… выскакивает ошибка: порядок сортировки не поддерживается базой данных
{: .prompt-warning }

Добавляем сервис в автозагрузки и проверяем доступность порта `5432`

```sh
$ sudo systemctl enable --now postgrespro-std-12
$ ss -nltup
```

## Настройка Postgres PRO

Разрешаем авторизацию пользователям из нашей сети

```sh
$ sudo nano /var/lib/pgpro/std-12/data/pg_hba.conf
...
#IPv4 local connections:
host all all 127.0.0.1/32 md5
host all all 192.168.11.0/24 md5
```

Создадим пароль для пользователя `postgres`

```sh
$ sudo su - postgres
$ psql
=# ALTER USER postgres WITH ENCRYPTED PASSWORD 'vTH886v4g2TqcD';
=# \q
$ exit
```

Перезапускаем сервис `postgrespro-std-12`

```sh
$ sudo systemctl restart postgrespro-std-12
```

## Установка сервера 1C

Для начала необходимо скачать дистрибутив server 1c под linux в каталог /tmp  
Сделать это можно с официального сайта, либо поискать в интернете

Распаковываем архив с дистрибутивом и устанавливаем

```sh
$ cd /tmp
$ tar xvf rpm64_8_3_17_1549.tar.gz
$ sudo dnf -y localinstall *.rpm
```

Меняем владельца и группу директории `/opt/1C`

```sh
$ sudo chown -R usr1cv8:grp1cv8 /opt/1C
```

Добавляем сервис srv1cv83 в автозагрузку, запускаем его и проверяем статус

```sh
$ sudo systemctl enable srv1cv83
$ sudo systemctl start srv1cv83
$ sudo systemctl status srv1cv83
```

## Настройка сервера 1C

Создаем каталог, в котором будут храниться конфигурации 1с для подключения к базе

```sh
$ sudo mkdir -p /mnt/1c/base
$ sudo chown -R usr1cv8:grp1cv8 /mnt/1c/base
```

Редактируем конфигурационный файл сервера 1с `srv1cv83`, указываем путь к новому каталогу

```sh
$ sudo nano /etc/sysconfig/srv1cv83
...
SRV1CV8_DATA=/mnt/1c/base
```

Перезапускаем сервис `srv1cv83` и проверяем статус

```sh
$ sudo systemctl restart srv1cv83
$ sudo systemctl status srv1cv83
```

## Установка и настройка драйвера HASP

Устанавливаем необходимую утилиту

```sh
$ sudo dnf -y install glibc
```

Скачиваем rpm-пакеты

```sh
$ cd /tmp
$ wget http://download.etersoft.ru/pub/Etersoft/HASP/last/x86_64/CentOS/7/haspd-7.90-eter2centos.x86_64.rpm
$ wget http://download.etersoft.ru/pub/Etersoft/HASP/last/x86_64/CentOS/7/haspd-modules-7.90-eter2centos.x86_64.rpm
```

Устанавливаем их

```sh
$ sudo dnf -y localinstall haspd*
```

Настраиваем

```sh
$ sudo nano /etc/haspd/hasplm.conf
...
NHS_IP_LIMIT = 127.0.0.1, 192.168.11.0/24
```

В этой строчке перечислены сети и хосты, которые смогут видеть HASP-ключ

Перезапускаем сервис `haspd`, смотрим статус

```sh
$ sudo systemctl restart haspd
$ sudo systemctl status haspd
```

## Настройка Firewall

Открываем порты

```sh
$ sudo firewall-cmd --permanent --add-port=80/tcp
$ sudo firewall-cmd --permanent --add-port=1540/tcp
$ sudo firewall-cmd --permanent --add-port=1541/tcp
$ sudo firewall-cmd --permanent --add-port=1560/tcp
$ sudo firewall-cmd --permanent --add-port=5432/tcp
$ sudo firewall-cmd --reload
$ sudo firewall-cmd --list-all
```

## Создание базы 1C

На Windows-машине запускаем консоль администрирования серверов 1С Предприятия  
Правой кнопкой мыши (ПКМ):

```
Central 1C:Enterprise 8.3 servers - Создать - Центральный сервер 1С:Предприятие 8.3
```

![](/assets/img/posts/2020/10/08/1c-step1.png){: w="300" }
_Создать - Центральный сервер 1С_

```
Протокол: TCP
Имя: server1c
IP порт: 1540
```

![](/assets/img/posts/2020/10/08/1c-step2.png){: w="300" }
_Параметры центрального сервера_

Кластер `Local cluster` при этом будет создан автоматически

![](/assets/img/posts/2020/10/08/1c-step3.png){: w="300" }
_Параметры кластера_

Теперь создаем информационную базу

```
Выбираем "Local cluster" - Информационные базы - ПКМ - Создать - Информационная база
```

![](/assets/img/posts/2020/10/08/1c-step4.png){: w="300" }
_Создать информационную база_

```
Имя: base1c
Защищенное соединение: выключено
Сервер баз данных: sever1c
Тип СУБД: PostgreSQL
База данных: base1c
Пользователь сервера БД: postgres
Пароль пользователя БД: vTH886v4g2TqcD (этот пароль был задан на этапе установки БД)
Создать базу данных в случае ее отсутствия: +
```

![](/assets/img/posts/2020/10/08/1c-step5-2.png){: w="300" }
_Параметры информационной базы_

## Установка шрифтов для подготовки к публикации web-сервера

Установка необходимых пакетов

```sh
$ sudo dnf -y install rpm-build ttmkfdir fontconfig freetype libgsf unixODBC
```

Так же нам нужен пакет `cabextract`, но под Centos 8 в базовых репозиториях его нет. По-этому скачиваем его из стороннего источника и устанавливаем

```sh
$ cd /tmp
$ wget https://pkgs.dyn.su/el8/base/x86_64/cabextract-1.9-2.el8.x86_64.rpm
$ sudo dnf -y localinstall cabextract-1.9-2.el8.x86_64.rpm
```

Скачиваем файл спецификации для установки шрифтов microsoft

```sh
$ wget http://corefonts.sourceforge.net/msttcorefonts-2.5-1.spec
```

Подготавливаем пакет шрифтов

```sh
$ rpmbuild -bb msttcorefonts-2.5-1.spec
```

При выполнении команды `rpmbuild` должны скачаться все шрифты, и собраться пакет. Если в процессе выполнения команды появится ошибка, например: `Connection timed out, не удалось разрешить адрес зеркала`, нужно запустить команду еще раз.

Устанавливаем пакет шрифтов

```sh
$ sudo rpm -ivh $HOME/rpmbuild/RPMS/noarch/msttcorefonts-2.5-1.noarch.rpm
```

## Установка web-сервера Apache

Устанавливаем Apache

```sh
$ sudo dnf -y install httpd
```

Добавляем его в автозагрузку, запускаем и смотрим статус

```sh
$ sudo systemctl enable --now httpd
$ sudo systemctl status httpd
```

Создадим каталог, он будет использован как путь публикации для web-сервера 1с

```sh
$ sudo mkdir -p /var/www/infobase
```

Создадим пустой файл, он будет указан в качестве конфигурационного файла web-сервера 1с

```sh
$ sudo touch /etc/httpd/conf.d/base.conf
```

Далее публикуем базу 1С

```sh
$ cd /opt/1C/v8.3/x86_64
$ sudo ./webinst -apache24 -wsdir base -dir /var/www/infobase/ -connStr "Srvr=server1c;Ref=base1c;" -confPath /etc/httpd/conf.d/base.conf
Publication successful
```

где
- `dir` — путь к папке веб сервера, ранее созданная директория
- `connStr` — путь к расположению файловой базы 1С
- `confPath` — путь к файлу конфигурации веб сервера, ранее созданный файл (должен быть быть пустым)
- `publish` - указывает необходимое действие, в данном случае публикацию, может быть опущен, так как это действие по умолчанию
- `wsdir` - имя публикации, по которому к базе следует обращаться из браузера, обратите внимание, что оно регистрозависимое
- `connStr` - строка соединения, состоит из нескольких частей: `Srvr` - имя сервера, `Ref` - имя базы на сервере, каждая часть должна заканчиваться служебным символом `;`

Меняем владельца и группу созданного файла, перезапускаем Apache

```sh
$ sudo chown apache:apache /var/www/infobase/default.vrd
$ sudo systemctl restart httpd
```

## Настройка SELinux

Создаем файл с описанием политик web 1с для Selinux

```sh
$ cd /tmp
$ nano httpd_1c.te
module httpd_1c 1.0;
require {
type httpd_t;
type httpd_tmp_t;
type user_home_t;
type httpd_sys_content_t;
class dir { add_name create read remove_name rmdir write };
class file { create lock open read rename setattr unlink write };
class file execute;
}
============= httpd_t ==============
!!!! This avc is allowed in the current policy
allow httpd_t httpd_sys_content_t:file write;
!!!! This avc is allowed in the current policy
allow httpd_t user_home_t:dir { add_name create read remove_name rmdir write };
allow httpd_t user_home_t:file rename;
!!!! This avc is allowed in the current policy
allow httpd_t user_home_t:file { create lock open read setattr unlink write };
!!!! This avc can be allowed using the boolean ‘httpd_tmp_exec’
allow httpd_t httpd_tmp_t:file execute;
```

Компилируем и установим политику

```sh
$ sudo checkmodule -M -m -o httpd_1c.mod httpd_1c.te
$ sudo semodule_package -o httpd_1c.pp -m httpd_1c.mod
$ sudo semodule -i httpd_1c.pp
```

Перезапустим сервер Apache

```sh
$ sudo systemctl restart httpd
```

В моем случае верхнее правило не помогло, пришлось поступать следующим образом:

Анализируем лог, компилируем и устанавливаем еще одну политику

```sh
$ cd /tmp
$ sudo grep httpd /var/log/audit/audit.log | grep denied | audit2allow -m httpdlocalconf > httpdlocalconf.te
$ sudo grep httpd /var/log/audit/audit.log | grep denied | audit2allow -M httpdlocalconf
$ sudo semodule -i httpdlocalconf.pp
```

Проверяем в браузере:

```
http://192.168.11.235/base
```

Или через тонкий клиент 1С по тому же адресу.

На этом установка Сервера 1с с базой данных PostgreSQL и публикацией сервера в web завершена. Можно подключать USB-ключ с лицензией к серверу и работать.  
Но, если вы разворачиваете ради тестирования, можно установить эмулятор HASP.

## Установка эмулятора HASP в Centos 8 из исходников

Устанавливаем утилиты сборки

```sh
$ sudo dnf -y install gcc gcc-c++ make
```

Устанавливаем заголовки ядра

```sh
$ sudo dnf -y install kernel-devel
```

Устанавливаем утилиты для сборки зависимостей

```sh
$ sudo dnf -y install jansson-devel libusb.i686 elfutils-libelf-devel
```

Устанавливаем GIT

```sh
$ sudo dnf -y install git
```

Скачиваем исходники `VHCI_HCD`, `LIBUSB_VHCI` и `USB_HASP` в каталог `/usr/src`

```sh
$ cd /usr/src
$ sudo wget https://sourceforge.net/projects/usb-vhci/files/linux%20kernel%20module/vhci-hcd-1.15.tar.gz/download -O vhci-hcd-1.15.tar.gz
$ sudo wget https://sourceforge.net/projects/usb-vhci/files/native%20libraries/libusb_vhci-0.8.tar.gz/download -O libusb_vhci-0.8.tar.gz
$ sudo git clone https://github.com/sam88651/UsbHasp.git
```

Распаковываем исходники `VHCI_HCD` и `LIBUSB_VHCI`

```sh
$ sudo tar -xpf libusb_vhci-0.8.tar.gz
$ sudo tar -xpf vhci-hcd-1.15.tar.gz
```

Компилируем `VHCI_HCD`

```sh
$ KVER=`uname -r`
$ cd vhci-hcd-1.15
$ sudo mkdir -p linux/${KVER}/drivers/usb/core
$ sudo cp /usr/src/kernels/${KVER}/include/linux/usb/hcd.h linux/${KVER}/drivers/usb/core
$ sudo sed -i 's/#define DEBUG/\/\/#define DEBUG/' usb-vhci-hcd.c
$ sudo sed -i 's/#define DEBUG/\/\/#define DEBUG/' usb-vhci-iocifc.c
$ sudo sed -i 's/VERIFY_READ, //' usb-vhci-iocifc.c
$ sudo sed -i 's/VERIFY_WRITE, //' usb-vhci-iocifc.c
$ sudo make KVERSION=${KVER}
```

Устанавливаем `VHCI_HCD`

```sh
$ sudo make install
```

Загружаем модуль `usb_vhci_hcd`

```sh
$ echo "usb_vhci_hcd" | sudo tee /etc/modules-load.d/usb_vhci.conf
$ sudo modprobe usb_vhci_hcd
```

Загружаем модуль `usb_vhci_iocifc`

```sh
$ echo "usb_vhci_iocifc" | sudo tee -a /etc/modules-load.d/usb_vhci.conf
$ sudo modprobe usb_vhci_iocifc
```

Компилируем `LIBUSB_VHCI`

```sh
$ cd ../libusb_vhci-0.8
$ sudo ./configure
$ sudo make -s
```

Устанавливаем `LIBUSB_VHCI`

```sh
$ sudo make install
$ echo "/usr/local/lib" | sudo tee /etc/ld.so.conf.d/libusb_vhci.conf
$ sudo ldconfig
```

Компилируем `UsbHasp`

```sh
$ cd ../UsbHasp
$ sudo make -s
```

Устанавливаем `UsbHasp`

```sh
$ sudo cp dist/Release/GNU-Linux/usbhasp /usr/local/sbin
```

Создаем директорию для дампов usb-ключей

```sh
$ sudo mkdir /etc/usbhaspkey/
```

Создаем системный unit `usbhaspemul.service`

```sh
$ sudo nano /etc/systemd/system/usbhaspemul.service
[Unit]
Description=Emulation HASP key for 1C
Requires=haspd.service
After=haspd.service
[Service]
Type=simple
ExecStart=/usr/bin/sh -c 'find /etc/usbhaspkey -name "*.json" | xargs /usr/local/sbin/usbhasp'
Restart=always
[Install]
WantedBy=multi-user.target
```

Добавляем службу `usbhaspemul` в автозагрузку

```sh
$ sudo systemctl daemon-reload
$ sudo systemctl enable usbhaspemul
```

Загружаем дампы usb-ключей в каталог `/etc/usbhaspkey/` (дампы искать в интернете)

```sh
$ sudo cp /tmp/Dumps/1c_server_x64.json /etc/usbhaspkey/
$ sudo cp /tmp/Dumps/100user.json /etc/usbhaspkey/
```

Пробуем запустить USB HASP Emulator, проверяем статус

```sh
$ sudo systemctl start usbhaspemul
$ sudo systemctl status usbhaspemul
```

## Разное

Сервер разворачивался в VirtualBox, параметры:

```
OS: Centos 8.2 dvd iso
сеть: сетевой мост
$ cat /etc/hosts
192.168.11.235 server1c

В винде в drivers/etc/hosts
192.168.11.235 server1c
```{: .prompt-info }```
