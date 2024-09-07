---
title: "SSH-авторизация по ключу (сертификату) в Centos"
date: "2019-10-02"
categories: 
  - Manuals
tags: 
  - "centos"
  - "ssh"
  - "ssh-copy-id"
  - "ssh-keygen"
coverImage: "password-strength-in-2019-two-factor-authentication.png"
image:
  path: /commons/password-strength-in-2019-two-factor-authentication.png
  alt: "SSH-авторизация по ключу"
---

> **SSH-ключи** (Secure Shell keys) - это пары криптографических ключей, используемых для аутентификации и шифрования данных при подключении к удаленным серверам по протоколу SSH (Secure Shell). Каждая пара состоит из двух частей:
> - Открытый ключ (public key): доступен всем и используется для шифрования данных при отправке запроса на сервер.
> - Приватный ключ (private key): хранится только на локальном компьютере и используется для расшифровки данных, полученных от сервера.
{: .prompt-tip }

Есть виртуальная инфраструктура на базе Proxmox, где один виртуальный сервер выполняет роль основного сервера (`dev`, `nginx-proxy`) и имеет внешний IP. И есть куча дополнительный виртуальных серверов.

**Задача:** необходимо настроить возможность авторизоваться с dev-сервера на другие сервера по внутренним IP адресам без ввода пароля.

Подключаемся к dev-серверу и создаем открытый и закрытый ключи

```sh
[root@localhost]# ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:rHLydfgdfgdfgcvbcvbcvbG3GM9tosDSERPfsEW8 root@ct-dev-01
The key's randomart image is:
+---[RSA 2048]----+
|^/o. .           |
|*=@=o o          |
|.*+*o+ E         |
|+.+ o o.         |
|+o.  .  S        |
|+.     .         |
|..  o.o.         |
|  . .=. .        |
|  .o  ..         |
+----[SHA256]-----+
```

В результате выполнения команды сгенерировалось 2 файла в каталоге `~/.ssh/`

- `id_rsa.pub` — публичный ключ
- `id_rsa` — секретный ключ

Копируем наш публичный ключ на сервер, к которому мы будем подключаться без ввода пароля

```sh
$ sudo ssh-copy-id root@192.168.12.32
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/root/.ssh/id_rsa.pub"
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
root@192.168.12.32's password: 

Number of key(s) added: 1

Now try logging into the machine, with: "ssh 'root@192.168.12.32'"
and check to make sure that only the key(s) you wanted were added.
```

Проверяем:

```sh
$ sudo ssh root@192.168.12.32
Last login: Wed Oct  2 09:48:44 2019 from 192.168.12.2
$ ls -la .ssh
total 9
-rw------- 1 root root 396 Oct  2 09:48 authorized_keys
-rw-r--r-- 1 root root 193 Sep 19 10:47 known_hosts
```

На сервере, которому мы передали публичный ключ, появился файл `authorized_keys`. Содержимое этого файла и есть содержимое публичного ключа.
Таким образом, используя команду `ssh-copy-id` можно передать публичный ключ всем серверам, к которым в последствии мы будем подключаться.
