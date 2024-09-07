---
title: "Добавляем функцию поиска по сайту рядом с меню в мобильной версии сайта для Wordpress темы Ultra"
date: "2017-10-10"
categories: 
  - Content-Management
tags: 
  - "mobile"
  - "ultra"
  - "wordpress"
image:
  path: /commons/1003248_1dd0_2.jpg
  alt: "Добавляем функцию поиска по сайту"
---

> **WordPress** - это свободно распространяемая система управления содержимым сайта с открытым исходным кодом, написанная на PHP и использующая сервер базы данных MySQL. Она выпущена под лицензией GNU GPL версии 2.
{: .prompt-tip }

Заходим в админку > Внешний вид > Настроить

Выбираем пункт `Дополнительные стили`

Добавляем следующие строки:

```
/* для мобильных устройств (максимальная ширина здается в настройках темы) */
@media (max-width: 1024px){

/* отображаем иконку поиска */
nav#site-navigation.main-navigation .menu-search {
display: block;
left: -38px; 
}

/* скрываем иконку поиска при нажатии на конпку "Меню" */
nav#site-navigation.main-navigation.toggled .menu-search {
 display: none; 
}
}
```