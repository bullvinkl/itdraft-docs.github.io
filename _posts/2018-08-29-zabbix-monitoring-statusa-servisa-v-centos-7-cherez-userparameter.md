---
title: "Zabbix - мониторинг статуса сервиса в Centos 7 через UserParameter"
date: "2018-08-29"
categories: 
  - Monitoring-System
tags: 
  - "centos"
  - "userparameter"
  - "zabbix"
image:
  path: /commons/cloud-3406627_1280.jpg
  alt: "мониторинг статуса сервиса"
---

> **Zabbix** — свободная система мониторинга и отслеживания статусов разнообразных сервисов компьютерной сети, серверов и сетевого оборудования, написанная Алексеем Владышевым. Для хранения данных используется MySQL, PostgreSQL, SQLite или Oracle Database, веб-интерфейс написан на PHP.
{: .prompt-tip }

![](/assets/img/posts/2018/08/29/pic-2018-08-29_12-01_zabbix.png){: w="300" }

## Настраиваем сервер, который собираемся мониторить

[Устанавливаем zabbix-agent на сервер]({% post_url 2017-10-10-ustanovka-i-obnovlenie-zabbix-agent-na-centos-7 %})

Добавляем `UserParameter` в файл `zabbix_agentd.conf` и перезапускаем `Zabbix Agent`

```sh
$ sudo nano /etc/zabbix/zabbix_agentd.conf
### Option: UserParameter
#	User-defined parameter to monitor. There can be several user-defined parameters.
#	Format: UserParameter=<key>,<shell command>
#	See 'zabbix_agentd' directory for examples.
#
# Mandatory: no
# Default:
# UserParameter=

UserParameter=systemd.unit.is-active[*],systemctl is-active --quiet '$1' && echo 1 || echo 0
UserParameter=systemd.unit.is-failed[*],systemctl is-failed --quiet '$1' && echo 1 || echo 0
UserParameter=systemd.unit.is-enabled[*],systemctl is-enabled --quiet '$1' && echo 1 || echo 0

$ sudo systemctl restart zabbix-agent
```

Из значений UserParameter видно, что в дальнейшем можно мониторить:

- активный ли сервис (`is-active`)
- не завершился ли он с ошибкой (`is-failed`)
- добавлен ли он в автозагрузку (`is-enabled`)

## Настраиваем Zabbix Server

Переходим в web-интерфейс Zabbix Server

Добавляем сервер, который собираемся мониторить, в новый узел сети:
```
Настройка - Узел сети - Создать узел сети
```

Создаем новый элемент данных:
```
Настройки — Узлы сети — выбираем нужный узел — Элементы данных — Создать элемент данных
```

![](/assets/img/posts/2018/08/29/pic-2018-08-29_12-08_zabbix-item.png){: w="300" }
_Создать элемент данных_

- Имя: `srv-ftp-01:vsftpd_status`
- Тип: `Zabbix агент`
- Ключ: `systemd.unit.is-active[vsftpd]`
- Тип информации: `Числовой (целое положительное)`
- Тип данных: `Десятичный`
- Интервал обновления (в сек): `60`
- Новая группа элементов данных: `Services`

Создаем новый триггер 
``` 
Настройки — Узлы сети — выбираем нужный узел — Триггеры — Создать триггер
```

![](/assets/img/posts/2018/08/29/pic-2018-08-29_12-09_zabbix-trigger.png){: w="300" }
_Создать триггер_

- Имя: `srv-ftp-01:vsftpd_status`
- Важность: `Высокая`
- Выражение: `{srv-ftp-01:systemd.unit.is-active[vsftpd].last(0)}=0`
- Описание: `Проверка статуса сервиса vsftpd - systemctl status vsftpd`
