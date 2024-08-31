---
title: "Используем формат изображений WebP в Wordpress для Nginx"
date: "2023-08-02"
categories: 
  - Wordpress
tags: 
  - nginx
  - plugins
  - webp
  - wordpress
image:
  path: /commons/e9c1804a-9ae6-421a-a3a7-f720b8200ab9.webp
  alt: "Используем формат изображений WebP в Wordpress для Nginx"
---

> **WebP** — формат сжатия изображений как с потерями, так и без потерь, предложенный компанией Google Inc. в 2010 году. Основан на алгоритме сжатия неподвижных изображений из видеокодека VP8.

## Подготовка Wordpress

Для автоматической конвертации jpg и png картинок в wordpress будем использовать плагин "WebP Express". Но сам плагин в не будет создавать дополнительную нагрузку на wordpress, т.к. включать опцию "Alter HTML" мы не будем.

Настройки плагина оставляем дефолтные, включив только конвертацию при загрузке

![](/assets/img/posts/2023/08/02/webpexp2.png)

![](/assets/img/posts/2023/08/02/webpexp3.png)

![](/assets/img/posts/2023/08/02/webpexp4.png)

Запускаем конвертацию картинок

![](/assets/img/posts/2023/08/02/webpexp1.png)

Таким образом, плагин WebP Express используется только для первоначальной конвертации картинок в webp формат и для авто конвертации при загрузки новых картинок.

Результат конвертации: картинки весят в 6 раз меньше

![](/assets/img/posts/2023/08/02/image-1.png)

## Настройка Nginx

Отредактируем конфиг Nginx
```sh
$ sudo nano /etc/nginx/conf.d/wp.conf
...
# WebP Express rules
    location ~* ^/?wp-content/.*\.(png|jpe?g)$ {
      add_header Vary Accept;
      expires 365d;
      if ($http_accept !~* "webp"){
        break;
      }
      try_files
        /wp-content/webp-express/webp-images/doc-root/$uri.webp
        $uri.webp
        /wp-content/plugins/webp-express/wod/webp-on-demand.php?xsource=x$request_filename&wp-content=wp-content
        ;
    }

    # Route requests for non-existing webps to the converter
    location ~* ^/?wp-content/.*\.(png|jpe?g)\.webp$ {
        try_files
          $uri
          /wp-content/plugins/webp-express/wod/webp-realizer.php?xdestination=x$request_filename&wp-content=wp-content
          ;
    }
# WebP Express rules ends
...
```

Проверим на ошибки и перечитаем конфиг
```sh
$ sudo nginx -t
$ sudo nginx -s reload
```

Смотрим на результат

![](/assets/img/posts/2023/08/02/webpexp5-1024x425.png)

![](/assets/img/posts/2023/08/02/webpexp6-1024x425.png)

Как видно по второму скриншоту, теперь nginx подменяет jpg/png картинки на webp