---
layout: post
title: "[Решено] Cloudflare - TOO MANY REDIRECTS"
date: "2024-09-27"
categories:
  - Manuals
tags:
  - cloudflare
image:
  path: /commons/GettyImages-950753248.webp
  alt: "Cloudflare - ERR_TOO_MANY_REDIRECTS"
---

> ERR_TOO_MANY_REDIRECTS - это ошибка, возникающая в браузере, когда сайт попадает в бесконечный цикл перенаправлений. Это означает, что сайт постоянно перенаправляет пользователя на другой адрес, а тот, в свою очередь, перенаправляет на первый адрес, и так далее, создавая неограниченный цикл редиректов.
> В браузере (например, в Chrome) эта ошибка может выводиться как “ERR_TOO_MANY_REDIRECTS” или “This webpage has a redirect loop problem”. Пользователь не сможет доступа к содержимому сайта, пока не будет найдено и исправлено причину зацикливания редиректов.
{: .prompt-info }

При смене DNS провайдера на Cloudflare и включении опции Proxied сайт перестал загружаться, браузер показывал ошибку `ERR_TOO_MANY_REDIRECTS`.

Для исправления ошибки необходимо установлено значение `Full` для `SSL/TLS encryption`.
По умолчанию Cloudflare выставляет значение `Flexible`, предполагая, что будет использоваться HTTP для подключения к источнику. В нашем случае, сайт расположен на GitHub Pages, к нему уже подключен SSL сертификат. GitHub перенаправлением весь трафик на HTTPS, не понимая, что исходный клиент уже использует HTTPS. По этому, это перенаправление создает бесконечный цикл.

Для исправления ошибки переходим:

```
 SSL/TL > Overview > SSL/TLS encryption > Configure
```

![](/assets/img/posts/2024/09/27/ssl-tls.png){: w="300" }
_SSL/TLS encryption_

Выбираем слкдующую опцию:

```
Custom SSL/TLS > Full
```

![](/assets/img/posts/2024/09/27/ssl-tls-full.png){: w="300" }
_Custom SSL/TLS_

Более подробно можно изучить [на сайте Cloudflare](https://developers.cloudflare.com/ssl/troubleshooting/too-many-redirects/#full-or-full-strict-encryption-mode){:target="_blank"}
