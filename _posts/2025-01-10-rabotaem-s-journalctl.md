---
layout: post
title: "Работаем с Journalctl"
date: "2025-01-10"
categories:
  - Logging-System
tags:
  - journalctl
image:
  path: /commons/working-on-computer.jpg
  alt: "Работаем с Journalctl"
---
> **journalctl** — это инструмент командной строки, используемый для просмотра логов, которые собирает демон journald в systemd. Логи хорошо проиндексированы и структурированы таким образом, что позволяет системным администраторам легко анализировать их и управлять ими на основе различных параметров, таких как фильтрация логов по времени, последовательности загрузки, конкретной службе, серьезности и т. д.
{: .prompt-tip }

## Просмотр журнала за определенный период времени

С определенной даты и времени:
```bash
journalctl --since "2024-12-18 06:00:00"
```

С определенной даты и по определенное дату и время
```bash
journalctl --since "2024-12-17" --until "2024-12-18 10:00:00
```

Со вчерашнего дня:
```bash
journalctl --since yesterday
```

С 9 утра и до момента, час назад:
```bash
journalctl --since 09:00 --until "1 hour ago"
```

## Просмотр сообщений ядра

```bash
journalctl -k
```

## Просмотр журнала логов для определенного сервиса systemd или приложения

Просмотреть логи от NetworkManager
```bash
journalctl -u NetworkManager.service
```

Если нужно найти название сервиса, используйте команду:
```bash
systemctl list-units --type=service
```

Просмотреть все сообщения от nginx за сегодня
```bash
journalctl /usr/sbin/nginx --since today
```

Или указав конкретный PID:
```bash
journalctl _PID=1
```

## Дополнительные опции просмотра

Следить за появлением новых сообщений (аналог `tail -f`):
```bash
journalctl -f
```

Открыть журнал «перемотав» его к последней записи:
```bash
journalctl -e
```

## Удаление журналов

Удалить журналы, оставив только последние 100 Мб:
```bash
journalctl --vacuum-size=100M
```

Удалить журналы, оставив журналы только за последние 7 дней:
```bash
journalctl --vacuum-time=7d
```
