---
title: "Мониторинг определенной службы Windows машины в Zabbix"
date: "2015-08-01"
categories: 
  - Monitoring-System
tags: 
  - "service"
  - "windows"
  - "zabbix"
image:
  path: /commons/pexels-christina-morillo-1181244-scaled.jpg
  alt: "Мониторинг службы"
---

> **Zabbix** — свободная система мониторинга и отслеживания статусов разнообразных сервисов компьютерной сети, серверов и сетевого оборудования, написанная Алексеем Владышевым. Для хранения данных используется MySQL, PostgreSQL, SQLite или Oracle Database, веб-интерфейс написан на PHP.
{: .prompt-tip }

Находим наш узел

- Items -> Create new  

Задаем имя, в строке `Ключ` пишем:  

- `service_state[имя службы]` (имя брать из свойства службы в строке "имя службы", например `service_state[ArcGIS License Manager]`)  
- Интервал обновления: `60 сек`  
- Период хранения: `7 дней`

Сохраняем.  
  
Создаем триггер: Задаем имя, в строке выражение пишем: `{arcgisserver:service_state[ArcGIS License Manager].last(0)}=6`  
где:  

- `6` - остановлен. Т.е. отсылать алерт когда служба остановлена  
- `arcgisserver` - имя нашего узла  
- важность - `высокая`

Ключи:

- 0 - запущен  
- 1 - пауза  
- 2 - ожидание старта  
- 3 - ожидание паузы  
- 4 - ожидание продолжения  
- 5 - ожидание остановки  
- 6 - остановлен  
- 7 - неизвестно  
- 255 - такой службы не существует

Сохраняемся.  
Так же можно добавить график для наглядности.
