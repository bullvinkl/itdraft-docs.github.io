---
title: "Добавляем рейтинг к записи с помощью плагина WP-PostRatings"
date: "2017-04-04"
categories: 
  - Wordpress
tags: 
  - "wordpress"
  - "wp-postratings"
image:
  path: /commons/7592.png
  alt: "Добавляем рейтинг к записи с помощью плагина WP-PostRatings"
---

> WordPress - это свободно распространяемая система управления содержимым сайта с открытым исходным кодом, написанная на PHP и использующая сервер базы данных MySQL. Она выпущена под лицензией GNU GPL версии 2.
{: .prompt-tip }

Чтобы добавить рейтинг к записи, и разместить этот рейтинг под заголовком:

- Скачиваем и устанавливаем плагин WP-PostRatings  
- На странице настройки плагина выставляем нужные параметры  
- Вставляем отображение плагина в отдельной записи, для этого редактируем файл `single.php`

Для шаблона Ultra  
Ultra: Отдельная запись (`single.php`)

Ищем строчку

```
<h1 class="entry-title">
```

и после неё добавляем

```
<?php if(function_exists('the_ratings')) { the_ratings(); } ?>
```

Получим следующий вид:

```
<div class="container">
<h1 class="entry-title"><?php echo get_the_title(); ?></h1><?php ultra_breadcrumb(); ?>
<?php if(function_exists('the_ratings')) { the_ratings(); } ?>
</div><!-- .container -->
```
