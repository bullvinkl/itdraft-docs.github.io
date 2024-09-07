---
title: "Установка Oracle SQL Developer в Ubuntu 16.04"
date: "2016-08-18"
categories: 
  - Database-System
tags: 
  - sqldeveloper
  - oracle
  - ubuntu
image:
  path: /commons/ethereum_2-0_3-min.png
  alt: "Установка Oracle SQL Developer"
---

> **Oracle SQL Developer** - это бесплатная интегрированная среда разработки (IDE) для Oracle Database, которая позволяет разработчикам и администраторам баз данных создавать, тестировать и оптимизировать SQL-скрипты и PL/SQL-код.
{: .prompt-tip }

Устанавливаем Java JDK

```sh
$ sudo apt-get install default-jdk
```

Скачиваем архив `sqldeveloper-4.1.3.20.78-no-jre.zip` с сайта Oracle (Other Platforms)

Переносим архив в другую директорию и переходим в эту директорию

```sh
$ sudo mv sqldeveloper*.zip /usr/local/bin
$ cd /usr/local/bin
```

Распаковываем архив

```sh
$ sudo unzip sqldeveloper-4.1.3.20.78-no-jre.zip
```

Делаем файл `sqldeveloper.sh` исполняемым

```sh
$ sudo chmod +x /usr/local/bin/sqldeveloper/sqldeveloper.sh
```

Создаем сим линк

```sh
$ sudo ln -s /usr/local/bin/sqldeveloper/sqldeveloper.sh /bin/sqldeveloper
```

Удаляем архив

```sh
$ sudo rm /usr/local/bin/sqldeveloper*.zip
```

Редактируем содержимое скрипта `sqldeveloper.sh`

```sh
$ sudo nano /usr/local/bin/sqldeveloper/sqldeveloper.sh
```

Меняем

```sh
#!/bin/bash
cd "dirname $0"/sqldeveloper/bin && bash sqldeveloper $*
```

на

```sh
#!/bin/bash
cd /usr/local/bin/sqldeveloper/sqldeveloper/bin && bash sqldeveloper $*
```

При первом запуске `sqldeveloper` надо указать полный путь до установленного JDK.

Чтобы узнать путь, надо перейти в каталог и посмотреть установленные версии

```sh
$ cd /usr/lib/jvm
$ ls
```

В моем случае это `default-java java-1.8.0-openjdk-amd64 java-8-openjdk-amd64`

Таким образом полный путь будет: `/usr/lib/jvm/java-1.8.0-openjdk-amd64`

Запускаем `sqldeveloper`

```sh
$ sqldeveloper
```

На запрос
- `Type the full pathname of a JDK installation (or Ctrl-C to quit), the path will be stored in /home/user/.sqldeveloper/4.1.0/product.conf`

вводим
- `/usr/lib/jvm/java-1.8.0-openjdk-amd64`

Чтоб запускать sqldeveloper не через терминал а значком приложения создадим файл

```sh
$ sudo nano /usr/share/applications/sqldeveloper.desktop
[Desktop Entry]
Exec=sqldeveloper
Terminal=false
StartupNotify=true
Categories=GNOME;Oracle;
Type=Application
Icon=/usr/local/bin/sqldeveloper/icon.png
Name=SQL Developer
```

Затем надо выполнить команду

```sh
$ sudo update-desktop-database
```
