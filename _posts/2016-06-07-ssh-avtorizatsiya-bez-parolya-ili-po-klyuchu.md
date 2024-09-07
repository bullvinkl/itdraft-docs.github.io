---
title: "SSH авторизация без пароля или по ключу"
date: "2016-06-07"
categories: 
  - Security-System
tags: 
  - centos
  - ssh
  - putty
  - windows
image:
  path: /commons/evillimiter-featured.png
  alt: "SSH авторизация по ключу"
---

> **SSH** — протокол удаленного администрирования, обеспечивающий безопасное управление операционными системами и туннелирование TCP-соединений.
{: .prompt-tip }

На локальной машине (OS Linux) генерируем ключ

```sh
$ ssh-keygen -t rsa -b 2048 -f /home/user/.ssh/id_rsa -N ''
Generating public/private dsa key pair.
Your identification has been saved in /home/user/.ssh/id_rsa.
Your public key has been saved in /home/user/.ssh/id_rsa.pub.
The key fingerprint is:
95:e8:94:83:74:5c:63:0a:e1:4d:6d:77:30:86:aa:7b user@u1zer
The key's randomart image is:
+--[ RSA 2048]----+
| +ooo+.+. |
| o *.=++... |
| o Boo. . |
| o.o |
| .S |
| . |
| . |
| . E |
| . |
+-----------------+
```

где
- `-t rsa` - тип шифрования
- `-b 2048` - длина
- `-f /home/user/.ssh/id_rsa ` - каталог где будет сохранен ключ id_rsa и его публичный ключ id_rsa.pub
- `-N ''` - позволяет указать ключевую фразу в строчке, в данном случае парольная фраза пустая

Получаем два файла `id_rsa` (приватный ключ) и `id_rsa.pub` (публичный ключ).

Копируем файл `id_rsa.pub` на сервер, куда мы будем подключаться

```sh
$ ssh-copy-id -i ~/.ssh/id_rsa.pub user@192.168.1.96
```

Также убедитесь что на сервере разрешена авторизация по ключу, для этого в файле `/etc/ssh/sshd_config` должны быть строки:

```
RSAAuthentication yes
PubkeyAuthentication yes
AuthorizedKeysFile .ssh/authorized_keys
```

Если на сервере вообще хотим запретить авторизацию по паролям, выставляем параметр в `/etc/ssh/sshd_config`

```
PasswordAuthentication no
PermitEmptyPasswords no
```

После всего этого можно и сделать проверку на локальной машине:

```sh
$ ssh user@192.168.1.96
```

На локальной Windows-машине надо утилитой `puttygen` сконвертировать открытый ключ в свой формат, и в настройках `putty` указать этот сконвертированный файл
