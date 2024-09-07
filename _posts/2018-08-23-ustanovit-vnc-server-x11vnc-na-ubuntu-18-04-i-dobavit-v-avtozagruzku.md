---
title: "Установить VNC-сервер x11vnc в Ubuntu 18.04 и добавить в автозагрузку"
date: "2018-08-23"
categories: 
  - Linux
tags: 
  - "ubuntu"
  - "vnc"
  - "x11vnc"
image:
  path: /commons/978728_0af3_5.jpg
  alt: "VNC-сервер x11vnc в Ubuntu"
---
> **X11VNC** - это сервер удаленного доступа к графическому интерфейсу Linux, работающий по протоколу VNC. Он позволяет получать доступ к уже запущенному сеансу работы X-сервера с любого удаленного компьютера, используя любого клиента VNC.
{: .prompt-tip }

Устанавливаем x11vnc

```sh
$ sudo apt-get install x11vnc
```

Создаем пароль на подключение

```sh
$ mkdir /home/user/.vnc/
$ sudo x11vnc -storepasswd /home/user/.vnc/passwd
Enter VNC password: 
Verify password:    
Write password to /home/user/.vnc/passwd?  [y]/n 
Password written to: /home/user/.vnc/passwd
```

Для создания сервиса и добавления его в автозагрузку, создаем файл

```sh
$ sudo nano /lib/systemd/system/x11vnc.service

[Unit]
Description=Start x11vnc at startup.
After=multi-user.target
[Service]
Type=simple
ExecStart=/usr/bin/x11vnc -auth /home/user/.Xauthority -forever -loop -noxdamage -repeat -rfbauth /home/user/.vnc/passwd -rfbport 5900 -shared -display :1 -ncache 10 -bg -o /var/log/x11vnc.log
[Install]
WantedBy=multi-user.target
```

Добавляем созданные файл в автозагрузку и стартуем

```sh
$ sudo systemctl daemon-reload
$ sudo systemctl enable x11vnc.service
$ sudo systemctl start x11vnc.service
```
