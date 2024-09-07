---
title: "Интеграция KeePass с SSH (SSH-Key), RDP, SFTP клиентами"
date: "2021-02-21"
categories: 
  - Manuals
tags: 
  - "keepass"
  - "rdp"
  - "sftp"
  - "ssh"
image:
  path: /commons/website-3374825_1280.jpg
  alt: "Интеграция KeePass с SSH (SSH-Key), RDP, SFTP клиентами"
---

> **KeePass Password Safe** — кроссплатформенная свободная программа для хранения паролей, распространяемая по лицензии GPL
{: .prompt-tip }

Для интеграции KeePass используется настройки URL Overrides

## Подключение из KeePass по RDP без ввода IP, логина, пароля

Так как дефолтный RDP-клиент `mstsc` не поддерживает передачу логина/пароля из командной строки, воспользуемся прослойкой в виде утилиты [Remote Desktop Plus](https://www.donkz.nl/). Скачиваем и устанавливаем её.

Переходим к настройке KeePass

```
Сервис – Параметры - Интеграция – Переопределение URL 
```

![](/assets/img/posts/2021/02/21/kee1.png){: w="300" }

![](/assets/img/posts/2021/02/21/kee2.png){: w="300" }

Добавляем новую схему:

```
- Схема: rdp
- Переопределение: cmd://rdp /v:{URL:RMVSCM} /u:{USERNAME} /p:{PASSWORD}
```

![](/assets/img/posts/2021/02/21/kee4.png){: w="300" }

Применяем изменения, переходим в KeePass и добавляем новую запись или изменяем ранее добавленную.

В поле «URL-ссылка» прописываем запись в виде:

```
rdp://%ip%:%port%
```

![](/assets/img/posts/2021/02/21/kee5.png){: w="300" }

Теперь если выделить эту запись и нажать сочетание клавишь Ctrl+U откроется RDP-сессия, ip/логин/пароль подтянется из KeePass.

## Подключение из KeePass по SSH без ввода IP, логина, пароля

Скачиваем [Putty](https://the.earth.li/~sgtatham/putty/latest/w64/putty-64bit-0.74-installer.msi) и устанавливаем его, что бы в системе прописались нужные переменные.

Если пользуетесь Kitty (форк Putty), переименовываем `putty.exe` в `_putty.exe` в директории `C:\Program Files\PuTTY`, скачиваем [Kitty](https://github.com/cyd01/KiTTY/releases), переименовываем `kitty.exe` в `putty.exe` и перемещаем его в `C:\Program Files\PuTTY`

Переходим к настройке KeePass

```
Сервис – Параметры - Интеграция – Переопределение URL 
```

В блоке "Встроенные переопределения" отключаем схему "ssh" и добавляем новую схему:

```
- Схема: ssh
- Переопределение: cmd://"putty.exe" -ssh {USERNAME}@{URL:RMVSCM} -P {T-REPLACE-RX:/{URL:PORT}/^-1$/22/} -pw "{PASSWORD}"
```

![](/assets/img/posts/2021/02/21/kee6.png){: w="300" }

Применяем изменения, переходим в KeePass и добавляем новую запись или изменяем ранее добавленную.

В поле `URL-ссылка` прописываем запись в виде:

```
ssh://%ip%:%port%
```

Причем если ssh-порт стандартный (22/tcp), то его можно не указывать.

![](/assets/img/posts/2021/02/21/kee7.png){: w="300" }

Теперь если выделить эту запись и нажать сочетание клавиш Ctrl+U откроется SSH-сессия в Putty либо Kitty, ip/логин/пароль подтянется из KeePass.

## Подключение из KeePass по SSH, используя SSH-ключ (сертификат), который хранится в KeePass

Скачиваем плагин для KeePass - [KeeAgent](https://lechnology.com/software/keeagent/#download)

Распаковываем его в `C:\Program Files (x86)\KeePass Password Safe 2\Plugins` и перезапускаем KeePass.

Переходим к настройке KeePass

```
Сервис – Параметры - KeeAgent
```

Кроме отмеченных по умолчанию пунктов, отмечаем:

- Always require user confirmation when a client program requests to use a key
- Enable agent for Windows OpenSSH (experimental)

Отмечаем интеграции (предварительно создать каталог `C:\Temp\`):

- Create Cygwin compatible socket file  
    `Path: C:\Temp\cyglockfile`
- Create msysGit compatible socket file  
    `Path: C:\Temp\syslockfile`

![](/assets/img/posts/2021/02/21/kee8.png){: w="300" }

Применяем изменения, переходим в KeePass и добавляем новую запись или изменяем ранее добавленную.

Переходим во вкладку `Дополнительно`, и в разделе `Прикрепляемые файлы` добавляем приватный ssh-ключ, который был сгенерирован утилитой PuttyGen. (Если у вас не настроена авторизация по ssh-ключу, в интернете много статей по настройке)

![](/assets/img/posts/2021/02/21/kee9.png){: w="300" }

Переходим во вкладку KeeAgent и отмечаем:

- Allow KeeAgent to use this entry

![](/assets/img/posts/2021/02/21/kee10.png){: w="300" }

Применим изменения, переходим в KeePass, выделяем добавленную запись и нажимаем `Ctrl+U`

Откроется новая SSH-сессия в Putty либо Kitty, ssh-ключ подтянется из KeePass.

Так же появится запрос об использовании SSH-Ключа, а после подключения - уведомление от KeeAgent.

![](/assets/img/posts/2021/02/21/kee11.png){: w="300" }

![](/assets/img/posts/2021/02/21/kee12.png){: w="300" }

Причем теперь можно подключаться по SSH с использованием SSH-ключа из сторонних SSH-клиентов (Windows OpenSSH, Putty, Kitty, mRemoteNG) и SSH-ключ будет подтягиваться из KeePass.

![](/assets/img/posts/2021/02/21/kee13.png){: w="300" }

Если KeePass заблокирован, то будет появляться окно ввода пароля для KeePass

![](/assets/img/posts/2021/02/21/kee14.png){: w="300" }

К сожалению, SSH-клиенты xShell и SecureCRT не поддерживаются для использования схемы keepass+ssh-ключ, т.к. они используют свои агенты для хранения ssh-ключей.

## Подключение из KeePass по SFTP при помощи WinSCP

Скачиваем и устанавливаем [WinSCP](https://winscp.net/eng/download.php)

Переходим к настройке KeePass

```
Сервис – Параметры - Интеграция – Переопределение URL
```

Добавляем новую схему:

```
- Схема: scp
- Переопределение: cmd://"{ENV_PROGRAMFILES_X86}\WinSCP\WinSCP.exe" {BASE:SCM}://{USERNAME}:{PASSWORD}@{BASE:HOST}:{T-REPLACE-RX:/{BASE:PORT}/-1//}{BASE:PATH}
```

![](/assets/img/posts/2021/02/21/kee15.png){: w="300" }

Применяем изменения, переходим в KeePass и добавляем новую запись или изменяем ранее добавленную.

В поле `URL-ссылка` прописываем запись в виде:

```
scp://%ip%:%port%
```

![](/assets/img/posts/2021/02/21/kee16.png){: w="300" }

Теперь если выделить эту запись и нажать сочетание клавиш `Ctrl+U`, откроется SFTP-сессия в SFTP-клиенте WinSCP, ip/логин/пароль подтянется из KeePass. А если настроена авторизация по SSH-ключу, то из KeePass подтянется SSH-ключ.

Таким же образом можно настроить интеграцию KeePass с другими протоколами / клиентами: FTP, VNC и др.
