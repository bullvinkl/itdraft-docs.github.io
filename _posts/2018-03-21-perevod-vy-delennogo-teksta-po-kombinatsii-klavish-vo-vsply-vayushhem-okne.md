---
title: "Перевод выделенного текста по комбинации клавиш во всплывающем окне"
date: "2018-03-21"
categories: 
  - Manuals
tags: 
  - "hotkey"
  - "translate"
  - "ubuntu"
  - "zenity"
image:
  path: /commons/1051518_28b0_3.jpg
  alt: "Перевод выделенного текста по комбинации клавиш во всплывающем окне"
---

> **Zenity** - утилита для вывода диалоговых окон GTK+ из командной строки и скриптов командной оболочки. Она является переисанной версией программы gdialog, адаптированной для среды GNOME.
{: .prompt-tip }

Ставим софт

```sh
$ sudo apt install translate-shell gawk curl mplayer less aspell zenity xsel
```

Создаем скрипт

```sh
$ nano /home/user/.translate_textbox

#!/usr/bin/env bash
a=`xsel -o | trans :ru -no-ansi -b`
# файл с переводом
tmp="/tmp/gtrans"
# файл с переводом 2, см. дальшше
tmp2="/tmp/gtrans2"
echo -e "$a" > $tmp
# из-за ошибки в программе trans, удаляем последние 5 символов и записываем результат в др. файл
rev $tmp | cut -c 6- | rev > $tmp2
# Выводим
zenity --text-info --width="500" --height="300" --title="Перевод" --filename=$tmp2
```

Делаем его исполняемым

```sh
$ chmod +x /home/user/.translate_textbox
```

Назначаем горячие клавиши: Настройки - Устройства - Клавиатура

- Имя: `trabslate box`
- Команда: `/home/user/.translate_textbox`
- Комбинация клавиш: `ctrl + 1`
