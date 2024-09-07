---
title: "Zabbix. Мониторинг логов"
date: "2018-06-21"
categories: 
  - Monitoring-System
tags: 
  - "log"
  - "zabbix"
image:
  path: /commons/digital-enablement.jpg
  alt: "Мониторинг логов"
---

> **Zabbix** — свободная система мониторинга и отслеживания статусов разнообразных сервисов компьютерной сети, серверов и сетевого оборудования, написанная Алексеем Владышевым. Для хранения данных используется MySQL, PostgreSQL, SQLite или Oracle Database, веб-интерфейс написан на PHP.
{: .prompt-tip }

Требуется мониторить логи Centos 7 `/var/log/messages` на предмет записи `Out of memory: Kill process`

## Подготовка сервера, который мы будем мониторить

Добавляем файл `/var/log/messages` в группу `zabbix` и назначить на него права доступа 640

```sh
$ sudo chown root:zabbix /var/log/messages
$ sudo chmod 0640 /var/log/messages
```

Проверяем настройки zabbix agent.

```sh
Server = ip-адрес Zabbix Server
ServerActive = ip-адрес Zabbix Server
Hostname = Такое же имя, как прописано в вэб интерфейсе Zabbix Server (Настройки - Узлы сети)
```

## Настройка мониторинга в Zabbix

```
Настройки - Узлы сети - выбираем наш сервер, на котором мы будем мониторить логи
```

Добавляем новую группу элементов данных:  
Переходим во вкладку `Группы элементов данных`, нажимаем `Создать группу элементов данных`, я назвал ее `log`.

Добавляем новый элемент данных:  
Переходим во вкладку `Элементов данных`, нажимаем `Создать элемент данных`

- Имя: `log_messages`
- Тип: `Zabbix агент (активные)`
- Ключ: `log[/var/log/messages,"Out of memory: Kill process","UTF-8",100]`
- Тип информации: `Журнал (лог)`
- Группы элементов данных: `log`

Добавляем новый триггер:  
Переходим во вкладку `Триггеры`, нажимаем `Создать триггер`

- Имя: `00sr-server-01`
- Важность: `Высокая`
- Выражение: `{00sr-server-01:log[/var/log/messages,"Out of memory: Kill process","UTF-8",100].str(Kill,30)}=1 and {00sr-server-01:log[/var/log/messages,"Out of memory: Kill process","UTF-8",100].nodata(30)}=0`

Триггер срабатывает при наличии строки Kill в пересылаемых данных и отключается через 30 секунд отсутствия новых данных.

## UPD 03.04.2019

Бывают случаи, когда в элементах данный узла сети висит сообщение `zabbix cannot open file /var/log/messages 13 permission denied`

И из-за этого элемент данных не активен, следовательно логи не приходят на почту

Это случается из-за следующих моментов:

- SELinux блокирует zabbix-agent
- права доступа на файл /var/log/messages принадлежат пользователю root

Что бы добавить `zabbix-agent` в исключение для SELinux, устанавливаем утилиту `policycoreutils`

```sh
$ sudo yum install policycoreutils-python
```

И выполняем команду

```bash
$ sudo semanage permissive -a zabbix_agent_t
```

После этого надо перезапустить zabbix-agent

```bash
$ sudo systemctl restart zabbix-agent
```

Что бы открыть доступ для zabbix-agent на чтение файла /var/log/messages, изменим права доступа

```sh
$ sudo chown root:zabbix /var/log/messages
```
