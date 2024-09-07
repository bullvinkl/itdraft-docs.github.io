---
title: "Ошибки MySQL: Unknown collation 'utf8mb4_unicode_ci', Unknown character set 'utf8mb4'"
date: "2018-08-07"
categories: 
  - Database-System
tags: 
  - "centos"
  - "linux"
  - "mysql"
image:
  path: /commons/template-1599665_1280.png
  alt: "Unknown collation, Unknown character"
---

> **MySQL** - это клиент-серверное программное обеспечение, предназначенное для управления реляционными базами данных. Её исходный код открыт, что означает, что разработчики могут свободно использовать и изменять код.
{: .prompt-tip }

Эти ошибки у меня появились после того, как я начал переносить dump базы данных с одного сервера (свежая версия MySQL) на другой (более старая версия MySQL).  
По-хорошему, надо обновить версию MySQL на свежую, но т.к. на сервере расположено еще несколько сайтов, выполним следующие манипуляции.

## Исправляем ошибку ERROR 1273 Unknown collation: 'utf8mb4_unicode_ci'

Открываем dump базы данных в текстовом редакторе и заменяем `utf8mb4_unicode_ci` на `utf8_general_ci`

## Исправляем ошибку ERROR 1115 Unknown character set: 'utf8mb4'

Открываем dump базы данных в текстовом редакторе и заменяем `utf8mb4` на `utf8`
