---
title: "Мониторинг количества исходящих писем Postfix с помощью Zabbix"
date: "2016-03-23"
categories: 
  - Monitoring-System
tags: 
  - "postfix"
  - "zabbix"
image:
  path: /commons/laptops.png
  alt: "Мониторинг количества исходящих писем Postfix с помощью Zabbix"
---

> **Zabbix** — свободная система мониторинга и отслеживания статусов разнообразных сервисов компьютерной сети, серверов и сетевого оборудования. 
{: .prompt-tip }

На почтовом сервере в конфигурационном файле `/etc/zabbix/zabbix_agent.conf` в самом конце добавляем пользовательский параметр

```
UserParameter=mail.queuesize,/usr/sbin/postqueue -p | tail -n 1 | awk '{ if ($5 == "") print "0"; else print $5; }'
```

В Zabbix выбираем нужный узел сети (наш почтовый сервер) и создаем элемент данных:

- Имя: `Mail.Queue`
- Тип: `Zabbix агент`
- Ключ: `mail.queuesize` (этот параметр мы прописали в zabbix_agent.conf )

Создаем новый триггер

- Имя: `Mail.Queue`
- Выражение: `{00sr-mb-01:mail.queuesize.avg(#5)}>25`
- Важность: `Высокая`

где `00sr-mb-01` - имя нашего узла сети

Триггер будет срабатывать, если в очереди исходящих писем больше 25
