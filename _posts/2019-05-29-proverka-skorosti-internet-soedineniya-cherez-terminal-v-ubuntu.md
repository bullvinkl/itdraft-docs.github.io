---
title: "Проверка скорости интернет соединения через терминал в Ubuntu"
date: "2019-05-29"
categories: 
  - Linux
tags: 
  - "speedtest"
  - "ubuntu"
image:
  path: /commons/recommendation_sys-min.png
  alt: "Проверка скорости интернет соединения через терминал в Ubuntu"
---

> **Speedtest CLI** - это официальный инструмент для измерения скорости интернет-соединения, разработанный командой Ookla. Он доступен для использования в командной строке и может быть полезен для системных администраторов, разработчиков и энтузиастов компьютеров.
{: .prompt-tip }

Для проверки скорости нашего интернет-канала воспользуемся утилитой `speedtest-cli`. Установим ёё

```sh
$ sudo apt install speedtest-cli
```

Примеры команд для запуска утилиты `speedtest-cli`

Подробная информация при тестировании скорости

```sh
$ speedtest
Retrieving speedtest.net configuration...
Testing from OOO DelaemZabory Ltd. (192.168.0.19)...
Retrieving speedtest.net server list...
Selecting best server based on ping...
Hosted by Megafon (Moscow) [0.12 km]: 4.093 ms
Testing download speed................................................................................
Download: 76.63 Mbit/s
Testing upload speed......................................................................................................
Upload: 21.72 Mbit/s
```

Вывод только необходимой информации

```sh
$ speedtest --simple
Ping: 3.583 ms
Download: 87.92 Mbit/s
Upload: 94.95 Mbit/s
```

Посмотреть полный список доступных серверов для тестирования скорости

```sh
$ speedtest --list
Retrieving speedtest.net configuration...
12824) Akado Telecom (Moscow, Russia) [0.12 km]
 3682) Rostelecom (Moscow, Russian Federation) [0.12 km]
 1907) MTS (Moscow, Russian Federation) [0.12 km]
 6386) Megafon (Moscow, Russian Federation) [0.12 km]
10366) Orange Business Services, Russia & CIS (Moscow, Russian Federation) [0.12 km]
 6827) MGTS (Moscow, Russian Federation) [0.12 km]
...
```

Тестирование скорости до сервера, выбранного из списка выше

```sh
$ speedtest --server 12824
Retrieving speedtest.net configuration...
Testing from OOO DelaemZabory Ltd. (192.168.0.19)...
Retrieving speedtest.net server list...
Retrieving information for the selected server...
Hosted by Akado Telecom (Moscow) [0.12 km]: 2.602 ms
Testing download speed................................................................................
Download: 85.77 Mbit/s
Testing upload speed......................................................................................................
Upload: 95.25 Mbit/s
```

Сгенерировать картинку с данными о интернет-соединении что бы в дальнейшем можно было поделиться ей

```sh
$ speedtest --share
Retrieving speedtest.net configuration...
Testing from OOO DelaemZabory Ltd. (192.168.0.19)...
Retrieving speedtest.net server list...
Selecting best server based on ping...
Hosted by Akado Telecom (Moscow) [0.12 km]: 2.567 ms
Testing download speed................................................................................
Download: 89.37 Mbit/s
Testing upload speed......................................................................................................
Upload: 68.33 Mbit/s
Share results: http://www.speedtest.net/result/8294113864.png
```

Полный список команд

```sh
$ speedtest-cli -h
```
