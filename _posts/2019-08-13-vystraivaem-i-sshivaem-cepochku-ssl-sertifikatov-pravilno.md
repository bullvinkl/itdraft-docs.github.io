---
title: "Выстраиваем и сшиваем цепочку SSL-сертификатов правильно"
date: "2019-08-13"
categories: 
  - Linux
tags: 
  - "apache"
  - "nginx"
  - "ssl"
image:
  path: /commons/1400240_c61f_3.jpg
  alt: "сшиваем цепочку SSL-сертификатов"
---

> Цепочка сертификатов SSL включает в себя сертификаты поручителей, подтверждающие валидность документа в целом. Структура CA Bundle имеет такой вид:
> - Сертификат домена
> - Сертификаты посредников, или промежуточный (Intermediate).
> - Корневой сертификат (Root). 
{: .prompt-tip }

Мы выпустили SSL-сертификат для сайта.  
Допустим нам предоставили следующие файлы:

- `domain.crt` - SSL-сертификат сайта
- `Intermediate2.crt` - Промежуточный SSL-сертификат 2
- `Intermediate1.crt` - Промежуточный SSL-сертификат
- `CARoot.crt `- Корневой SSL-сертификат

Так же у нас уже имелись файлы:

- `request.csr` - запрос на получение сертификата
- `private.key` - приватный ключ


Правильная цепочка SSL-сертификатов имеет структуру:

```
SSL-сертификат сайта —> Промежуточный SSL-сертификат -> Промежуточный SSL-сертификат 2 -> … —> Корневой SSL-сертификат
```

Создадим из присланных нам файлов правильную цепочку SSL-сертификатов. Открываем терминал и выполняем команду:

```sh
$ cd /etc/nginx/ssl
$ sudo su
# cat domain.crt Intermediate1.crt Intermediate2.crt CARoot.crt > domain.ca-bundle
```

Таким образом мы получили файл `domain.ca-bundle`, который содержит правильную цепочку SSL-сертификатов

Проверим. Смотрим хэш-суммы SSL-сертификат сайта, полученного сертификата, приватного ключа и запроса на получение сертификата

```sh
$ sudo openssl x509 -noout -modulus -in domain.crt | openssl md5
(stdin)= b67fe14bdd9336fc9aef28605e69b9a7
$ sudo openssl x509 -noout -modulus -in domain.ca-bundle | openssl md5
(stdin)= b67fe14bdd9336fc9aef28605e69b9a7
$ sudo openssl rsa -noout -modulus -in private.key | openssl md5
(stdin)= b67fe14bdd9336fc9aef28605e69b9a7
$ sudo  openssl req -noout -modulus -in request.csr | openssl md5
(stdin)= b67fe14bdd9336fc9aef28605e69b9a7
```
