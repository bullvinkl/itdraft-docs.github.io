---
title: "Интеграция iTop ITSM & CMDB и FreeIPA"
date: "2021-08-20"
categories: 
  - Asset-Management
  - Directory-Service
tags: 
  - "centos"
  - "cmdb"
  - "freeipa"
  - "itop"
  - "itsm"
  - "linux"
  - "rocky-linux"
image:
  path: /commons/computer.jpg
  alt: "Интеграция iTop ITSM & CMDB и FreeIPA"
---

> **iTop (IT Operational Portal)** — это веб-продукт с открытым исходным кодом, предназначенный для автоматизации ИТ-подразделений предприятий и сервис провайдеров. iTop разработан на основе лучших практик ITIL/ITSM и в то же время является достаточно гибким, чтобы адаптироваться к процессам вашей организации.
{: .prompt-tip }

- В предыдущей статье была рассмотрена [установка iTop ITSM & CMDB в Rocky Linux]({% post_url 2021-07-16-ustanovka-itop-itsm-cmdb-v-centos-8-ili-rocky-linux %}).

Для интеграции iTop и FreeIPA необходимо завести во FreeIPA сервисный аккаунт (например: `itopsv`), и отредактировать конфигурационный файл `config-itop.php`

```sh
$ sudo nano /opt/itop/web/conf/production/config-itop.php
...
        'timezone' => 'Europe/Moscow',
...
        'authent-ldap' => array (
                'host' => 'ldap://ipa.example.loc',
                'port' => 389,
                'default_user' => 'uid=itopsv,cn=users,cn=accounts,dc=example,dc=loc',
                'default_pwd' => 'mysuperpasswd',
                'base_dn' => 'cn=users,cn=accounts,dc=example,dc=loc',
                'user_query' => '(&(uid=%1$s))',
                'options' => array (
                  17 => 3,
                  8 => 0,
                ),
                'start_tls' => false,
                'debug' => true,
```

Далее в web-админке iTop в разделе `Управление конфигурациями` создаем новый контакт (тип: Персона)

![](/assets/img/posts/2021/08/20/image-1.png){: w="300" }
_Управление конфигурациями_

Заполняем необходимые поля

![](/assets/img/posts/2021/08/20/image-2.png){: w="300" }

Далее переходим в раздел `Инструменты администратора - Учетные записи` и создаем новую учетную запись (тип: Пользователь LDAP)

![](/assets/img/posts/2021/08/20/image-3.png){: w="300" }
_Инструменты администратора - Учетные записи_

В строке `Персона` надо выбрать пользователя (которого мы создавали на предыдущем шаге)  
В строке `Логин` прописываем логин из FreeIPA

![](/assets/img/posts/2021/08/20/image-4.png){: w="300" }

Если iTop не может получить пользователей FreeIPA, все ошибки отображаются в лог-файле `error.log`:

```sh
$ tail -f /opt/itop/web/log/error.log
```
