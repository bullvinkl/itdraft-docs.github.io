---
title: "Установка и настройка MRTG + SNMP на CentOS 6"
date: "2017-10-04"
categories: 
  - Linux
tags: 
  - "centos"
  - "mrtg"
  - "snmp"
image:
  path: /commons/978728_0af3_5.jpg
  alt: "MRTG + SNMP"
---

> **MRTG** (Multi Router Traffic Grapher) - это инструмент для визуализации трафика и мониторинга данных, написанный на Perl. Он получает данные от наблюдаемого оборудования по протоколу SNMP или с использованием скриптов, и отображает их в виде различных графиков.
{: .prompt-tip }

Ставим утилиты:

```sh
$ sudo yum install net-snmp net-snmp-utils net-snmp-devel zlib libpng gd mrtg
```

После установке имеем следующие конфигурационные файлы:

```
/etc/snmpd/snmpd.conf
/etc/mrtg/mrtg.cfg
/etc/cron.d/mrtg
/etc/httpd/conf.d/mrtg.conf
```

Редактируем конфиг SNMP /etc/snmpd/snmpd.conf

```sh
$ sudo nano /etc/snmpd/snmpd.conf
...
com2sec local localhost public
group MyRWGroup v1 local
group MyRWGroup v2c local
group MyRWGroup usm local
view all included .1 80
access MyRWGroup "" any noauth exact all all none
syslocation Russia
syscontact Root
```

Добавляем службу snmp в автозагрузку и стартуем

```sh
$ sudo chkconfig snmpd on
$ sudo service snmpd restart
```

Проверяем

```sh
$ sudo snmpwalk -v 1 -c public localhost IP-MIB::ipAdEntIfIndex
```

ответ дожен быть такого вида:

```
IP-MIB::ipAdEntIfIndex.123.xx.yy.zzz = INTEGER: 2
IP-MIB::ipAdEntIfIndex.127.0.0.1 = INTEGER: 1
```

Настраиваем MRTG  
создаем файл настроек `/etc/mrtg/mrtg.cfg`

```sh
$ sudo cfgmaker --global 'WorkDir: /var/www/mrtg' --output /etc/mrtg/mrtg.cfg public@localhost
```

Проверим содержимое `/etc/mrtg/mrtg.cfg`  
ищем там WorkDir и указываем корректный путь

```
WorkDir: /var/www/mrtg
```

Соддаем файл `index.html` на основе нашего конфигурационного файла `/etc/mrtg/mrtg.cfg`

```
indexmaker --output=/var/www/mrtg/index.html /etc/mrtg/mrtg.cfg
```

Проверяем Cron Tab

```sh
$ cat /etc/cron.d/mrtg
```

Содержимое должно быть таким:

```
*/5 * * * * root LANG=C LC_ALL=C /usr/bin/mrtg /etc/mrtg/mrtg.cfg --lock-file /var/lock/mrtg/mrtg_l --confcache-file /var/lib/mrtg/mrtg.ok
```

Проверяем, добавлен ли Cron Tab в автозагрузку:

```sh
$ sudo chkconfig --list crond
```

Вывод:

```
crond 0:выкл 1:выкл 2:вкл 3:вкл 4:вкл 5:вкл 6:выкл
```

Если Cron Tab не запущен и не добавлен в автозагрузку, исправляем это:

```sh
$ sudo chkconfig crond on
$ sudo service crond start
```

Настраиваем Apache `/etc/httpd/conf.d/mrtg.conf`

```sh
$ sudo nano /etc/httpd/conf.d/mrtg.conf
...
Alias /mrtg /var/www/mrtg

Order deny,allow
Deny from all
Allow from 127.0.0.1
Allow from ::1
# Allow from .example.com
```

127.0.0.1 меняем на ip, которому разрешено смотреть результат

Перезапускаем Apache

```sh
$ sudo service httpd restart
```

Готово, смотрим результат

```
http://ваш_ip/mrtg/
```
