---
title: "Установка почтового сервера iRedMail на CentOS 7. Часть 2. Боремся со спамом"
date: "2019-02-07"
categories: 
  - Linux
  - iRedMail
tags: 
  - "centos"
  - "dovecot"
  - "iredmail"
  - "spamassassin"
image:
  path: /commons/analytics.jpg
  alt: "Установка iRedMail"
---

> **Спам** (англ. spam) — массовая рассылка корреспонденции рекламного характера лицам, не выражавшим желания её получать. Распространителей спама называют спамерами.

## Настройка Dovecot

Подключаем плагин `imap_sieve` в Dovecot

```sh
$ sudo nano /etc/dovecot/dovecot.conf
protocol imap {
    mail_plugins = $mail_plugins imap_quota imap_acl imap_sieve
    ...
}
plugin {
   ...
    # Antispam
    sieve_plugins = sieve_imapsieve sieve_extprograms

    # From elsewhere to Spam folder
    imapsieve_mailbox1_name = Junk
    imapsieve_mailbox1_causes = COPY
    imapsieve_mailbox1_before = file:/var/vmail/sieve/report-spam.sieve

    # From Spam folder to elsewhere
    imapsieve_mailbox2_name = *
    imapsieve_mailbox2_from = Junk
    imapsieve_mailbox2_causes = COPY
    imapsieve_mailbox2_before = file:/var/vmail/sieve/report-ham.sieve

    sieve_pipe_bin_dir = /var/vmail/sieve
    sieve_global_extensions = +vnd.dovecot.pipe +vnd.dovecot.environment +vnd.dovecot.debug
}
```

Создаем скрипт `report-spam.sieve`

```sh
$ sudo nano /var/vmail/sieve/report-spam.sieve
require ["vnd.dovecot.debug", "vnd.dovecot.pipe", "copy", "imapsieve", "environment", "variables"];

debug_log "report_spam executed ${1}";

if environment :matches "imap.user" "*" {
  # to use a global user: 
  #set "username" “amavis”;
  set "username" "${1}";
}

pipe :copy "sa-learn-spam.sh" [ "${username}" ];
```

Создаем скрипт `report-ham.sieve`

```sh
$ sudo nano /var/vmail/sieve/report-ham.sieve
require ["vnd.dovecot.debug", "vnd.dovecot.pipe", "copy", "imapsieve", "environment", "variables"];

debug_log "report_ham executed ${1}";

if environment :matches "imap.mailbox" "*" {
  set "mailbox" "${1}";
}

if string "${mailbox}" "Trash" {
  stop;
}

if environment :matches "imap.user" "*" {
  # to use a global user: 
  #set "username" “amavis”;
  set "username" "${1}";
}

pipe :copy "sa-learn-ham.sh" [ "${username}" ];
```

Создаем исполняемые `sa-learn` файлы

```sh
$ sudo nano /var/vmail/sieve/sa-learn-spam.sh 
exec /usr/bin/sa-learn -u ${1} --spam
```

```sh
$ sudo nano /var/vmail/sieve/sa-learn-ham.sh
exec /usr/bin/sa-learn -u ${1} --ham
```

Меняем владельца файлов на vmail и делаем файлы исполняемые

```sh
$ sudo chown vmail:vmail /var/vmail/sieve/report-*
$ sudo chown vmail:vmail /var/vmail/sieve/sa-learn-*
$ sudo chmod +x /var/vmail/sieve/report-*
$ sudo chmod +x /var/vmail/sieve/sa-learn-*
```

## Настройка SpamAssassin

Настраиваем хранение базы SpamAssasin в MySQL, редактируем файл `local.cf`

```sh
$ sudo nano /etc/mail/spamassassin/local.cf
use_bayes          1
bayes_auto_learn   1
bayes_auto_expire  1

# Store bayesian data in MySQL
bayes_store_module Mail::SpamAssassin::BayesStore::MySQL
bayes_sql_dsn      DBI:mysql:sa_bayes:127.0.0.1:3306

# Store bayesian data in MySQL
#bayes_store_module Mail::SpamAssassin::BayesStore::PgSQL
#bayes_sql_dsn      DBI:Pg:database:sql_server:sql_port
#
bayes_sql_username %user%
bayes_sql_password %password%
```

где:

-`%user%` - имя пользователя, который имеет доступ к базе `sa_bayes`

-`%password%` - пароль

Эти данные мы заведем чуть позже

Проверим версию SpamAssassin и скачаем схему базы `spamassassin`

```sh
$ sudo spamassassin -V
SpamAssassin version 3.4.0
  running on Perl version 5.16.3

$ cd
$ wget http://svn.apache.org/repos/asf/spamassassin/tags/spamassassin_release_3_4_0/sql/bayes_mysql.sql
```

Отредактируем файл `bayes_mysql.sql`  
В нем надо `varchar(200)` поменять на `varchar(191)`, иначе при добавлении схемы в базу произойдет не полностью

Создадим базу `sa_bayes`, пользователя и пароль

```sh
$ mysql -uroot -p
> CREATE DATABASE sa_bayes;
> USE sa_bayes;
> SOURCE /home/bayes_mysql.sql;
> GRANT SELECT, INSERT, UPDATE, DELETE ON sa_bayes.* TO %user%@localhost IDENTIFIED BY '%password%';
> FLUSH PRIVILEGES;
> EXIT;
```

Не забываем менять `%user%` и `%password%` на свои значения

## Еще одна настройка Dovecot

Эта настройка нужна, чтобы письма, помеченные в Thunderbird меткой spam автоматически добавлялись в базу SpamAssassin

Редактируем файл `dovecot.sieve`

```sh
$ sudo nano /var/vmail/sieve/dovecot.sieve
require ["fileinto", "vnd.dovecot.debug", "vnd.dovecot.pipe", "copy", "environment", "variables"];

# rule:[Move Spam to Junk Folder]
if header :is "X-Spam-Flag" "YES"
{
    fileinto "Junk";
    set "username" "amavis";
    pipe :copy "sa-learn-spam.sh" [ "${username}" ];
}
```

У нас в организации в основном используется `pop3` протокол (с целью не загромождать свободное место на сервер), по-этому я отключил автоматическое перемещение писем, помеченных SPAM, в каталог почтового ящика Спам, т.к. pop3 не умеет работать с каталогами. Для отключения перемещения надо закомментировать строку:

```
#    fileinto "Junk";
```

Перезапускаем Dovecot и Amavis

```sh
$ sudo systemctl restart  dovecot
$ sudo systemctl restart amavisd
```

## Дополнительные настройки

Так же. для того, что бы pop3-пользователи могли участвовать в процессе обучения на spam/ham было заведено 2 почтовых ящика:  
- `spam@itdraft.ru` - ящик для перенаправления спама  
- `ham@itdraft.ru` - ящик для писем, которые ошибочно были помечены как спам

Добавим в `сrontab` задания на обучение `spamassassin` и очистку почтовых ящиков, указанных выше:

```sh
$ sudo nano /var/spool/cron/root
# SpamAssassin learn spam at 00:05 from mailbox spam@itdraft.ru
5   0   *   *   *   /usr/bin/sa-learn --spam /var/vmail/vmail1/itdraft.ru/s/p/a/spam-2019.02.01.13.15.26/Maildir/new/

# SpamAssassin learn ham at 00:06 from mailbox ham@itdraft.ru
6   0   *   *   *   /usr/bin/sa-learn --ham /var/vmail/vmail1/itdraft.ru/h/a/m/ham-2019.02.01.13.16.51/Maildir/new/

# Deleete messages from spam@itdraft.ru and ham@itdraft.ru
15  0   *   *   *   /bin/rm -rf /var/vmail/vmail1/itdraft.ru/s/p/a/spam-2019.02.01.13.15.26/Maildir/new/*
17  0   *   *   *   /bin/rm -rf /var/vmail/vmail1/itdraft.ru/h/a/m/ham-2019.02.01.13.16.51/Maildir/new/*
```

В данный момент механизм обучения через пересылки писем уже не используется, т.е. `spamassasin` обучается за счет того, что вся почта с несуществующих ящиков и с алиаса info@itdraft.ru перенаправляется в ящик postmaster@itdraft.ru, а этот почтовый ящик подключен в thunderbird по протоколу imap. Thunderbird сам автоматически помечает спам-письма соответствующей меткой, и благодаря этой пометки происходит обучение spamassassin

## Настройки Amavis

Для того, чтобы Amavis не удалял письма помеченные в теме письма как spam, отредактируем файл `amavisd.conf`

```sh
$ sudo nano /etc/amavisd/amavisd.conf

# SPAM
$final_spam_destiny = D_BOUNCE
```

Для того, чтобы активировать обучение SpamAssassin в Amavis, отредактируем файл `amavisd.conf`

```sh
$ sudo nano /etc/amavisd/amavisd.conf

$sa_tag_level_deflt  = -999;
$sa_tag2_level_deflt = 5;
$sa_kill_level_deflt = 6.3;
$sa_dsn_cutoff_level = 10;
```

Перезапускаем Amavis

```sh
$ sudo systemctl restart amavisd
```