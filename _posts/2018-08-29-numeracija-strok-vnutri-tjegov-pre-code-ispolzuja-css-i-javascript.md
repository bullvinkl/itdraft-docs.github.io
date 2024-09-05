---
title: "Нумерация строк внутри тэгов pre,code используя css и javascript"
date: "2018-08-29"
categories: 
  - Wordpress
tags: 
  - "css"
  - "javascript"
  - "line-number"
image:
  path: /commons/1003248_1dd0_2.jpg
  alt: "Нумерация строк внутри тэгов pre,code"
---

> Тег <pre> предназначен для отображения предварительно отформатированного текста, обычно кода, в виде, близком к оригинальному. Он позволяет сохранить пробелы и форматирование текста, как было указано в исходном коде.
> Тег <code> используется для выделения кода в HTML-документе. Он не изменяет форматирование текста, но позволяет отличить код от обычного текста.

Что бы добавить нумерацию строк, после тэгов `</code>` и `</pre>` нужно вставить javascript

```javascript
<script language="javascript">
(function() {
    var pre = document.getElementsByTagName('pre'),
        pl = pre.length;
    for (var i = 0; i < pl; i++) {
        pre[i].innerHTML = '<span class="line-number"></span>' + pre[i].innerHTML + '<span class="cl"></span>';
        var num = pre[i].innerHTML.split(/\n/).length;
        for (var j = 0; j < num; j++) {
            var line_num = pre[i].getElementsByTagName('span')[0];
            line_num.innerHTML += '<span>' + (j + 1) + '</span>';
        }
    }
})();
</script>
```

Чтобы нумерация шла слева, перед строками исходника, нужно настроить оформление через css, например

```css
body {
  background-color:white;
  padding:50px 50px;
}

pre {
  background-color:#eee;
  overflow:auto;
  margin:0 0 1em;
  padding:.5em 1em;
}

pre code, pre .line-number {
  font:normal normal 12px/14px "Courier New",Courier,Monospace;
  color:black;
  display:block;
}

pre .line-number {
  float:left;
  margin:0 1em 0 -1em;
  border-right:1px solid;
  text-align:right;
}

pre .line-number span {
  display:block;
  padding:0 .5em 0 1em;
}

pre .cl {
  display:block;
  clear:both;
}
```

На этой странице можно увидеть пример вывода исходного кода с нумерацией строк