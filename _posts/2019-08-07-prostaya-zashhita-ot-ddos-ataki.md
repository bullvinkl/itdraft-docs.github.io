---
title: "Простая защита от DDoS-атаки"
date: "2019-08-07"
categories: 
  - Linux
tags: 
  - "ddos"
  - "iptables"
image:
  path: /commons/986308_ac87_7.jpg
  alt: "защита от DDoS"
---

> **DDoS** (Distributed Denial of Service) — распределённая атака типа «отказ в обслуживании». Сетевой ресурс выходит из строя в результате множества запросов к нему, отправленных из разных точек. Обычно атака организуется при помощи бот-нетов.
{: .prompt-tip }

Для начала определяем количество соединений с 1 ip к примеру на 443 порт

```sh
$ netstat -ntu | grep ":443\ " | awk '{print $5}'| cut -d: -f1 | sort | uniq -c | sort -nr | more
    130 185.215.60.212
     19 94.140.142.86
     19 3.89.157.122
     17 62.148.156.22
     17 178.210.35.17
     16 217.26.165.15
     16 188.17.163.148
     15 213.5.120.34
     14 95.153.129.96
     14 94.25.229.95
```

Как видно из выдачи, с одного ip идет большое количество соединений.

Чтобы вручную не блокировать ip ботов, напишем bash-скрипт

```sh
$ nano /home/script/ddos.sh
#!/bin/sh
# Задаем путь к скриптам
mypath=/home/script

# Определяем все соединения на порт 443 и записываем лог find.log
netstat -ntu | grep ":443\ " | awk '{print $5}'| cut -d: -f1 | sort | uniq -c | sort -nr | grep -v "127.0.0.1" | grep -v "8.8.8.8" > $mypath/find.log

# Создаем DROP правила iptables, блокировать IP если количество коннектов 50 и больше. И сохраняем правила в bash-скрипт ban_ip.sh
awk '{if ($1 > 50) {print "/sbin/iptables -A INPUT -p tcp --dport 443 -s " $2 " -j DROP";}}' $mypath/ddos.iplist >> $mypath/ban_ip.sh

# Запускаем только что созданный скрипт блокировки IP атакующих
/bin/bash $mypath/ban_ip.sh

# Очищаем скрипт ban_ip.sh
cat /dev/null > $mypath/ban_ip.sh
```

Добавляем скрипт ddos.sh в крон

```sh
$ sudo crontab -e
...
# запускать скрипт раз в 5 минут
*/5 * * * * /home/script/ddos.sh

# амнистия раз в месяц
@monthly root iptables -F
```
