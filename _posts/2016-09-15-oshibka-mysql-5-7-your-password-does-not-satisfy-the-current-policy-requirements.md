---
title: "Ошибка MySQL 5.7: your password does not satisfy the current policy requirements"
date: "2016-09-15"
categories: 
  - Database-System
tags: 
  - "centos"
  - "mysql"
image:
  path: /commons/computer.jpg
  alt: "your password does not satisfy the current policy requirements"
---

> **MySQL** - это свободная реляционная система управления базами данных (СУБД), разработанная компанией MySQL AB и позднее поглощенная корпорацией Oracle. Она распространяется под двумя лицензиями: GNU General Public License и собственной коммерческой лицензией.
{: .prompt-tip }

При создании пользователя базы появляется ошибка `your password does not satisfy the current policy requirements`

Чтобы исправить эту ошибку надо подключиться к mysql

```sh
$ sudo mysql -u root -p
```

Смотрим настройки безопасности

```sh
> SHOW VARIABLES LIKE 'validate_password%';
+--------------------------------------+--------+
| Variable_name | Value |
+--------------------------------------+--------+
| validate_password_dictionary_file | |
| validate_password_length | 5 |
| validate_password_mixed_case_count | 1 |
| validate_password_number_count | 1 |
| validate_password_policy | MEDIUM |
| validate_password_special_char_count | 1 |
+--------------------------------------+--------+
```

Изменяем эти значения 

```sh
> SET GLOBAL validate_password_length = 5;
> SET GLOBAL validate_password_number_count = 0;
> SET GLOBAL validate_password_mixed_case_count = 0;
> SET GLOBAL validate_password_special_char_count = 0;
```
