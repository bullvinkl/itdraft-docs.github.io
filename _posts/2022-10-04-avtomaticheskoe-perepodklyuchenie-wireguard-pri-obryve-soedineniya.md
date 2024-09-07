---
title: "Автоматическое переподключение WireGuard при обрыве соединения"
date: "2022-10-04"
categories: 
  - Security-System
  - Automation
tags: 
  - "python"
  - "reconnect"
  - "systemd"
  - "wireguard"
image:
  path: /commons/156398bb67a3d4ee7ab97075a1137791.jpg
  alt: "Автоматическое переподключение WireGuard при обрыве соединения"
---

> Протокол **WireGuard VPN** разработан без сохранения состояния. Соединения рассматриваются как интерфейс — когда они активны, они всегда остаются активными. Если соединение с VPN-сервером потеряно, интернет-соединение перестает работать до тех пор, пока VPN-сервер снова не станет доступным. Как только VPN-сервер снова становится доступным, WireGuard повторно устанавливает VPN-соединение, и трафик снова начинает проходить.
{: .prompt-tip }

Иногда возникают ситуации, когда WireGuard теряет коннект к серверу, а соединение не восстанавливается. К сожалению из-за архитектурных особенностей у WireGuard нет механизма переподключения при обрыве соединения.

- За основу взят скрипт из [github репозитория](https://github.com/platofan23/python_wireguard_auto_reconnect)

- Так же есть [готовый модуль Python](https://pypi.org/project/wireguard-reconnect/), но я его не тестировал

Создаем каталог и немного отредактируем исходный скрипт, добавив логирование

```sh
$ sudo mkdir -p /opt/python_wireguard_auto_reconnect
$ sudo nano /opt/python_wireguard_auto_reconnect/Wireguard_Reconnect.py
#Imports
import os
import time
import logging
logging.basicConfig(level=logging.INFO, filename="wg_reconnect.log",filemode="w", format="%(asctime)s %(levelname)s %(message)s")
from pythonping import ping
#Failed Tries
failedTrys = 0
#None Stop Running
while(True):
   #Test Ping Sucessful
   if(ping('172.16.0.1')._responses[0].success):
      #print('Ping sucessful!')
      #logging.info('Ping sucessful!')
      #Reset Failed Tries
      failedTrys = 0
   #Test Ping failed
   else:
      #print('Ping fail! Restarting wireguard connection!')
      logging.warning('Ping fail! Restarting wireguard connection!')
      #Restart Wireguard-Interface w0
      os.system('systemctl stop wg-quick@wg0.service')
      time.sleep(5)
      os.system('systemctl start wg-quick@wg0.service')
      #Add one failed Try
      failedTrys = failedTrys + 1
   #More than 50 failed Tries make one 30 minutes timeout
   if(failedTrys >= 50):
      #Reset failed Tries
      failedTrys = 0
      #print('More than 50 failed Pings. Waiting 30 minutes for next ping!')
      logging.error('More than 50 failed Pings. Waiting 30 minutes for next ping!')
      #Sleep for 30 minutes
      time.sleep(1800)
   else:
      #print('Waiting 30 seconds for next ping!')
      #logging.info('Waiting 30 seconds for next ping!')
      #Wait 30 seconds for next ping
      time.sleep(30)
```

IP-адрес `172.16.0.1` - ip нашего WireGuard сервера

Устанавливаем необходимые модули Python

```sh
$ sudo pip3 install pythonping
$ sudo pip3 install sentry-sdk
```

Создаем Systemd Unit

```sh
$ sudo nano /etc/systemd/system/wgreconnect.service
[Unit]
Description=Wireguard Reconnect
After=multi-user.target

[Service]
Type=simple
Restart=always
ExecStart=/usr/bin/python3 Wireguard_Reconnect.py
WorkingDirectory=/opt/python_wireguard_auto_reconnect/
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```

Перечитываем Systemd Unit, добавляем сервис в автозагрузку, запускаем

```sh
$ sudo systemctl daemon-reload
$ sudo systemctl enable wgreconnect.service
$ sudo systemctl start wgreconnect.service
$ systemctl status wgreconnect.service
```
