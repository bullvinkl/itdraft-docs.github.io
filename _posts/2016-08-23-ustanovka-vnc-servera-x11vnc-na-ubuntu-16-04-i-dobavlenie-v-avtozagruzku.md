---
title: "Установка VNC-сервера x11vnc на Ubuntu 16.04 и добавление в автозагрузку"
date: "2016-08-23"
categories: 
  - Security-System
tags: 
  - "ubuntu"
  - "vnc"
  - "x11vnc"
image:
  path: /commons/computer.jpg
  alt: "Установка VNC-сервера x11vnc"
---

> **X11VNC** - это сервер удаленного доступа к графическому интерфейсу Linux, работающий по протоколу VNC. Он позволяет получать доступ к уже запущенному сеансу работы X-сервера с любого удаленного компьютера, используя любого клиента VNC.
{: .prompt-tip }

## Установка программного обеспечения

Устанавливаем x11vnc

```sh
$ sudo apt-get install x11vnc
```

Создаем пароль на подключение

```sh
$ sudo x11vnc -storepasswd /home/user/.vnc/passwd
```

Для проверки, запускаем программу в терминале

```sh
$ sudo x11vnc -auth guess -forever -loop -noxdamage -repeat -rfbauth /home/user/.vnc/passwd -rfbport 5900 -shared
```

## Добавление в автозагрузку

Создаем юнит файлы

```sh
$ sudo nano /lib/systemd/system/x11vnc.service
[Unit]
Description=Start x11vnc at startup.
After=multi-user.target

[Service]
Type=simple
ExecStart=/usr/bin/x11vnc -auth guess -forever -loop -noxdamage -repeat -rfbauth /home/user/.vnc/passwd -rfbport 5900 -shared

[Install]
WantedBy=multi-user.target
```

Перечитываем юнит-файлы, добавляем созданный юнит в автозагрузку и стартуем

```sh
$ sudo systemctl daemon-reload
$ sudo systemctl enable x11vnc.service
$ sudo systemctl start x11vnc.service
```

Смотрим статус:

```sh
$ sudo systemctl status x11vnc.service
```

## UPD 2017.04.07

Systemd unit `/lib/systemd/system/x11vnc.service`

```sh
[Unit]
Description=Start x11vnc at startup.
After=multi-user.target

[Service]
Type=simple
ExecStart=/usr/bin/x11vnc -auth /run/user/1000/gdm/Xauthority -forever -loop -noxdamage -repeat -rfbauth /home/user/.vnc/passwd -rfbport 5900 -shared -display :0 -ncache 10 -bg -o /var/log/x11vnc.log

[Install]
WantedBy=multi-user.target
```

где
- `-auth /run/user/1000/gdm/Xauthority`  - это значение получили после выполнения команды: `ps wwwwaux | grep auth`
- `-bg`  - фоновый режим
- `-o /var/log/x11vnc.log`  - писать лог в файл

тестировалось на Ubuntu Gnome 16.10
