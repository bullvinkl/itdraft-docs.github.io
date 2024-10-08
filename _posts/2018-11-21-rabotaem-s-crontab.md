---
title: "Работаем с crontab"
date: "2018-11-21"
categories: 
  - Manuals
tags: 
  - cron
  - crontab
image:
  path: /commons/1162000_fab6_2.jpg
  alt: "Работаем с crontab"
---

> **Cron** - это планировщик задач в операционной системе Linux, который позволяет запускать определенные скрипты или команды в нужное время, по расписанию. Это полезно для автоматизации обслуживания или администрирования системы.
{: .prompt-tip }

Основные команды

- `crontab -u %username%` - определяет пользователя чьи задачи будут просматриваться/редактироваться, отсутствие данного параметра устанавливает текущего пользователя;
- `crontab -l` - показывает список текущих задач;
- `crontab -e` - запускает редактор планировщика задач;
- `crontab -r` - удаляет все текущие задачи.

Синтаксис cron

```
    *    *   *     *        *        команда
|минута|час|день|месяц|день недели|
```

- `#` - комментарий (строки начинающиеся с данного символа не выполняются);
- `,` - перечисление значений (1,2,3,4);
- `/` - каждые n раз (*/n - каждые n, */5 - каждые 5, */2 - каждые 2);
- `-` - интервал значений (1-5 - с 1 до 5, 4-6 - с 4 до 6).

следовательно, следующие записи соответствуют следующим строкам:

- `0 5 * * *` - каждый день в 5:00;
- `*/10 * * * *` - каждые 10 минут;
- `0 0 1 1 *` - 1 января каждого года;
- `0 9 * * 1,3,5` - понедельник, среду и пятницу в 9 утра;
- `0 0 1 * *` - каждое 1-е число месяца.

Примеры

- `*/5 * * * * root /home/script.sh` - запускать команду каждые пять минут
- `0 */3 * * * root /home/script.sh` - запускать каждые три часа
- `0 12-16 * * * root /home/script.sh` - запускать команду каждый час с 12 до 16 (в 12, 13, 14, 15 и 16) 
- `0 12,16,18 * * * root /home/script.sh` - запускать команду каждый час в 12, 16 и 18 часов 
- `*/1 * * * * /usr/bin/php /home/test.php`   - запуск каждую минуту php-скрипта test.php
- `0 */1 * * * /usr/bin/perl /home/test.pl`     - запуск каждый час perl-скрипта test.pl
- `*/5 * * * * root /home/script.sh > /home/log.txt 2>&1` - запускать команду каждые пять минут и записать результат выполнения в лог (перезапись лога)
- `*/5 * * * * root /home/script.sh >> /home/log.txt 2>&1` - запускать команду каждые пять минут и записать результат выполнения в лог (не перезаписывать файл)
- `*/5 * * * * /home/edigaryev/test.sh > /home/edigaryev/test-$RANDOM.log 2>&1` - запускать команду каждые пять минут и записать результат выполнения в лог, при этом имя лога меняется `$RANDOM`
- `*/5 * * * * /home/edigaryev/test.sh > /home/edigaryev/test-$(date '+%Y-%m-%d_%H-%M-%S').log 2>&1` - запускать команду каждые пять минут и записать результат выполнения в лог, при этом имя лога меняется `[$(date '+%Y-%m-%d_%H-%M-%S')]`
