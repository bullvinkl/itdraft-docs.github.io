---
title: "Счетчик просмотров записи в Word Press: плагин Pageviews"
date: "2016-09-01"
categories: 
  - Content-Management
tags: 
  - pageviews
  - wordpress
  - fontawesome
image:
  path: /commons/computer.jpg
  alt: "Счетчик просмотров записи в Word Press"
---

> **WordPress** - это свободно распространяемая система управления содержимым сайта с открытым исходным кодом, написанная на PHP и использующая сервер базы данных MySQL. Она выпущена под лицензией GNU GPL версии 2.
{: .prompt-tip }

По умолчанию плагин `Pageviews` показывает количество просмотров записи в ее конце.  
Изменяем внешний вид и расположение, на примере этого сайта

Редактируем файл `functions.php`, добавив в самом низу следующие строки:

```php
add_action( 'after_setup_theme', function() {
add_theme_support( 'pageviews' );
});
```

Это отключает вывод счетчика в конце записи

Чтобы добавить вывод счетчика в нужном месте, открываем файл `content-single.php` (в теме Ultra), ищем строчку `<footer class="entry-footer">` и добавляем `<?php do_action( 'pageviews' ); ?>`

```php
<footer class="entry-footer">
<?php do_action( 'ultra_entry_main_bottom' ); ?>
<?php ultra_entry_footer(); ?><?php do_action( 'pageviews' ); ?>
</footer><!-- .entry-footer -->
```

Добавляем вывод иконки перед счетчиком. Для этого, редактируем файл стилей `style.css`  
Ищем строчки

```css
.entry-footer .edit-link:before {
content: "\f0f6"; }
```

и после них дописываем:

```css
.entry-footer .pageviews-placeholder:before {
content: "\f06e"; }
```

где `\f06e` - код нужной иконки [FontAwesome](https://fontawesome.com/icons)
