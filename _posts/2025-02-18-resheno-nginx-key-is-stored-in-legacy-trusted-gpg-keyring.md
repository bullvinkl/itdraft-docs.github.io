---
layout: post
title: "[Решено] Nginx: Key is stored in legacy trusted.gpg keyring (/etc/apt/trusted.gpg)"
date: "2025-02-18"
categories:
  - Web
tags:
  - nginx
image:
  path: /commons/DC79MkJWAAA1qod.jpg
  alt: "Key is stored in legacy trusted.gpg keyring (/etc/apt/trusted.gpg)"
---
> **trusted.gpg** — это файл, содержащий доверенные ключи GPG (GNU Privacy Guard). GPG — это инструмент для шифрования и электронного подписывания, использующий ассиметричное шифрование, основанное на двух ключах: приватном и публичном. Файл trusted.gpg хранит публичные ключи, которым вы доверяете, и которые вы можете использовать для проверки подписей и шифрования сообщений.
{: .prompt-tip }

Добавив репозиторий Nginx и обновив кэш, получил предупреждение:
> W: https://nginx.org/packages/mainline/debian/dists/bookworm/InRelease: Key is stored in legacy trusted.gpg keyring (/etc/apt/trusted.gpg), see the DEPRECATION section in apt-key(8) for details.
{: .prompt-warning }

```bash
$ sudo apt update
```

Вывод команды:
```bash
Hit:1 http://security.debian.org/debian-security bookworm-security InRelease
Hit:2 http://deb.debian.org/debian bookworm InRelease                                                                                 
Hit:3 http://deb.debian.org/debian bookworm-updates InRelease                                                                         
Get:4 https://nginx.org/packages/mainline/debian bookworm InRelease [2,878 B]                                                         
Hit:5 http://deb.debian.org/debian bookworm-backports InRelease                                                                       
Hit:6 https://repo.zabbix.com/zabbix/7.2/release/debian bookworm InRelease     
Hit:7 https://repo.zabbix.com/zabbix-tools/debian-ubuntu bookworm InRelease
Hit:8 https://repo.zabbix.com/zabbix/7.2/stable/debian bookworm InRelease
Get:9 https://nginx.org/packages/mainline/debian bookworm/nginx Sources [22.4 kB]
Get:10 https://nginx.org/packages/mainline/debian bookworm/nginx amd64 Packages [30.2 kB]
Fetched 55.5 kB in 2s (28.6 kB/s)   
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
All packages are up to date.
W: https://nginx.org/packages/mainline/debian/dists/bookworm/InRelease: Key is stored in legacy trusted.gpg keyring (/etc/apt/trusted.gpg), see the DEPRECATION section in apt-key(8) for details.
```

![Images](/assets/img/posts/2025/02/18/gpg1.png){: w="300" }
_apt update_

## Проблема
Проблема возникает в связи с тем, что публичный ключ репозитория хранится по устаревшему пути `/etc/apt/trusted.gpg`.
В моем случае - ошибка с ключом репозитория Nginx.

Выведем список сохранённых ключей
```bash
$ sudo apt-key list
```

Вывод команды:
```bash
Warning: apt-key is deprecated. Manage keyring files in trusted.gpg.d instead (see apt-key(8)).
/etc/apt/trusted.gpg
--------------------
pub   rsa2048 2011-08-19 [SC] [expires: 2027-05-24]
      573B FD6B 3D8F BC64 1079  A6AB ABF5 BD82 7BD9 BF62
uid           [ unknown] nginx signing key <signing-key@nginx.com>

pub   rsa4096 2024-05-29 [SC]
      8540 A6F1 8833 A80E 9C16  53A4 2FD2 1310 B49F 6B46
uid           [ unknown] nginx signing key <signing-key-2@nginx.com>

pub   rsa4096 2024-05-29 [SC]
      9E9B E90E ACBC DE69 FE9B  204C BCDC D8A3 8D88 A2B3
uid           [ unknown] nginx signing key <signing-key-3@nginx.com>
...
```

![Images](/assets/img/posts/2025/02/18/gpg2.png){: w="300" }
_apt-key list_

Удалим ключи из trusted.gpg
```bash
sudo apt-key --keyring /etc/apt/trusted.gpg del 7BD9BF62
sudo apt-key --keyring /etc/apt/trusted.gpg del B49F6B46
sudo apt-key --keyring /etc/apt/trusted.gpg del 8D88A2B3
```

где
- 7BD9BF62, B49F6B46, 8D88A2B3  - последние 8 символов каждого ключа

## Решение

При выполнении команды `apt-key` сразу указываем файл ключа (параметр `--keyring <путь к файлу>`).
```bash
wget --quiet -O - https://nginx.org/keys/nginx_signing.key | sudo apt-key --keyring /etc/apt/trusted.gpg.d/nginx.gpg add -
```

![Images](/assets/img/posts/2025/02/18/gpg3.png){: w="300" }
_apt update_
