---
title: "Добавить IP-адрес посетителя в отчет Яндекс Метрики"
date: "2018-11-14"
categories: 
  - Other
tags: 
  - "metrika"
  - "yandex"
image:
  path: /commons/913980_54f3_3.jpg
  alt: "Добавить IP-адрес посетителя в отчет Яндекс Метрики"
---

> **Яндекс Метрика** - бесплатный сервис веб-аналитики от Яндекс, который позволяет отслеживать и анализировать поведение посетителей на сайте, оценить эффективность рекламы и оптимизировать работу сайта.
{: .prompt-tip }

## Предварительная настройка

Для добавления IP-адреса посетителя сайта в отчет Яндекс метрики необходимо немного модифицировать код счетчика.

Воспользуемся сервисом [l2.io](https://l2.io), который позволяет получить ip-адрес.

Добавим скрипт до счетчика метрики

```html
<!-- получаем ip адрес -->
<script type="text/javascript" src="https://www.l2.io/ip.js?var=userip"></script>
```

Теперь нам остается передать параметр ip в отчет, для этого добавляем строчку `params:{'ip': userip}` в основной код метрики.

В результате получим код счетчика:

```html
<!-- получаем ip адрес -->
<script type="text/javascript" src="https://www.l2.io/ip.js?var=userip"></script>
<!-- Yandex.Metrika counter -->
<script type="text/javascript" >
    (function (d, w, c) {
        (w[c] = w[c] || []).push(function() {
            try {
                w.yaCounter99999999 = new Ya.Metrika2({
                    id:99999999,
                    clickmap:true,
                    trackLinks:true,
                    accurateTrackBounce:true,
                    webvisor:true,
                    params:{'ip': userip}
                });
            } catch(e) { }
        });

        var n = d.getElementsByTagName("script")[0],
            s = d.createElement("script"),
            f = function () { n.parentNode.insertBefore(s, n); };
        s.type = "text/javascript";
        s.async = true;
        s.src = "https://mc.yandex.ru/metrika/tag.js";

        if (w.opera == "[object Opera]") {
            d.addEventListener("DOMContentLoaded", f, false);
        } else { f(); }
    })(document, window, "yandex_metrika_callbacks2");
</script>
<noscript><div><img src="https://mc.yandex.ru/watch/99999999" style="position:absolute; left:-9999px;" alt="" /></div></noscript>
<!-- /Yandex.Metrika counter -->
```

где `99999999` - ваш ID из Яндекс метрики

## Выводим IP в отчетах

Переходим в Метрику - Вэбвизор, нажимаем `Настроить столбцы` и добавляем столбец `Параметры визитов`

![](/assets/img/posts/2018/11/14/2018-11-14_11-50.png){: w="300" }

В результате, через некоторое время, после захода новых посетителей сайта, в вэбвизоре будет следующая картина:

![](/assets/img/posts/2018/11/14/2018-11-14_11-52.png){: w="300" }

Так же, отчеты по IP адресам можно посмотреть:  
Отчеты - Стандартные отчеты - Содержание - Параметры визитов

![](/assets/img/posts/2018/11/14/2018-11-14_11-57.png){: w="300" }

![](/assets/img/posts/2018/11/14/2018-11-14_11-58.png){: w="300" }

## UPD 28.11.2018

Как подсказали в комментариях, сервис [l2.io](https://www.l2.io) работает с перебоями, из-за этого будем использовать другой способ определения ip

Необходимо перед счетчиком метрики в код страницы добавить javascript:

```html
<script type="text/javascript">
var yaParams = {ip_adress: "<? echo $_SERVER['REMOTE_ADDR'];?>"};
</script>
```

И в самом коде счетчика метрики (см. выше) вместо `params:{'ip': userip}` надо вставить `params:window.yaParams`

## UPD 06.06.2019

Заметил, что в Яндекс Метрике появился новый код счетчика, с поддержкой вэбвизора 2.0

Потестировал для него определение ip-адрес посетителя с помощью сервиса l2.io, ip определяется корректно  
Вот так будет выглядеть обновленный код счетчика:

```html
<!-- Get user ip -->
<script type="text/javascript" src="https://www.l2.io/ip.js?var=userip"></script>
<!-- /Get user ip -->
<!-- Yandex.Metrika counter -->
<script type="text/javascript" >
   (function(m,e,t,r,i,k,a){m[i]=m[i]||function(){(m[i].a=m[i].a||[]).push(arguments)};
   m[i].l=1*new Date();k=e.createElement(t),a=e.getElementsByTagName(t)[0],k.async=1,k.src=r,a.parentNode.insertBefore(k,a)})
   (window, document, "script", "https://mc.yandex.ru/metrika/tag.js", "ym");

   ym(12345678, "init", {
        clickmap:true,
        trackLinks:true,
        accurateTrackBounce:true,
        webvisor:true,
        params:{'ip': userip}
   });
</script>
<noscript><div><img src="https://mc.yandex.ru/watch/12345678" style="position:absolute; left:-9999px;" alt="" /></div></noscript>
<!-- /Yandex.Metrika counter -->
```

где `12345678` - номер счетчика. Его надо поменять на Ваш (в 2-х местах)
