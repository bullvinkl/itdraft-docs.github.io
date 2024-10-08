---
title: "Общий доступ к экрану в Ubuntu 18.04 / 18.10"
date: "2019-03-28"
categories: 
  - Manuals
tags: 
  - "ubuntu"
  - "vnc"
image:
  path: /commons/working-on-computer.jpg
  alt: "Общий доступ к экрану в Ubuntu"
---
> **Общий доступ к экрану** - это технология, позволяющая нескольким пользователям одновременно просматривать и взаимодействовать с содержимым экрана компьютера или устройства.
{: .prompt-tip }

В операционной системе Ubuntu есть штатный VNC сервер, который отключен по-умолчанию.

Для того, что бы его активировать откроем:

```
Настройки > Общий доступ > Общий доступ к экрану
```

![](/assets/img/posts/2019/03/28/wp_share_vnc1.png){: w="300" }
_Настройки_

Выставляем настройки в соответствии со скриншотом

Запускаем терминал и выполняем команду

```sh
$ gsettings set org.gnome.Vino require-encryption false
```

Таким образом мы отключили шифрование для vnc-соединения, которое включено по-умолчанию.

Теперь можно на другом ПК или телефоне запустить VNC-клиент и подключиться к нашему ПК, который мы только что настраивали
