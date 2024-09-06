---
title: "Отображение тэгов (меток) в ленте блога на Wordpress"
date: "2018-07-03"
categories: 
  - Wordpress
tags: 
  - "tag"
  - "wordpress"
image:
  path: /commons/template-1599665_1280.png
  alt: "Отображение тэгов (меток) в ленте блога на WordPress"
---

> **WordPress** - это свободно распространяемая система управления содержимым сайта с открытым исходным кодом. Она написана на языке программирования PHP и использует сервер базы данных MySQL или MariaDB.

На примере темы Ultra:  
В админке переходим: `Внешний вид - Редактор`.  
Для редактирования выбираем файл `loops/loop-thumbnail-left.php`

Ищем строку:

```php
<footer class="entry-footer">
 <!--?php ultra_entry_footer(); ?-->   
</footer><!-- .entry-footer -->
```

И меняем её на:

```php
<footer class="entry-footer">
<!--?php
// вставка тэгов
?-->
<!--?php if (has_tag()) : ?-->
    <!-- tags -->
    <span class="tags-links">
        <!--?php
        $tags = get_the_tags(get_the_ID());
        foreach ($tags as $tag) {
            echo '<a href="' . get_tag_link($tag--->term_id) . '">' . $tag->name . ', ';
        }
        ?>
    </span>
    <!-- end tags -->
<!--?php endif; ?-->
<!--?php
//конец вставки тэгов
?-->
   
 <!--?php ultra_entry_footer(); ?-->   
</footer><!-- .entry-footer -->
```

Нажимаем `Обновить файл`

UPD.  
Т.к. в варианте выше в конце последнего тэга добавлялась лишняя запятая, добавим функцию, чтобы убрать последний символ из массива

```php
<footer class="entry-footer">
<!--?php
// вставка тэгов
?-->
<!--?php if (has_tag()) : ?-->
    <!-- tags -->
    <span class="tags-links">
        <!--?php
        $tags = get_the_tags(get_the_ID());
 $result_names = '';
        foreach ($tags as $tag) {
            $result_names .= '<a href="' . get_tag_link($tag--->term_id) . '">' . $tag->name . ', ';
        }
        echo substr($result_names, 0, -2);
        ?>
    </span>
    <!-- end tags -->
<!--?php endif; ?-->
<!--?php
//конец вставки тэгов
?-->
 <!--?php ultra_entry_footer(); ?-->   
</footer><!-- .entry-footer -->
```
