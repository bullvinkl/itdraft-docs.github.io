---
title: "Как добавить Чат с компанией в поисковую выдачу Яндекса"
date: "2018-08-17"
categories: 
  - Manuals
tags: 
  - yandex
image:
  path: /commons/penguins.png
  alt: "Как добавить Чат с компанией"
---

> Яндекс предлагает функцию **чата с компанией** в результатах поиска, которая позволяет пользователям начать общение с представителями компании прямо на странице результатов поиска. Это достигается за счет интеграции чат-платформы с сервисом Яндекс.Диалоги.
{: .prompt-tip }

Не так давно Яндекс добавил в свою поисковую выдачу возможность задать вопрос в чате  
Выглядит это следующим образом:

![](/assets/img/posts/2018/08/17/pic-2018-08-17_14-37-02.png){: w="300" }
_Чат с компанией_

## Регистрация в сервисе, поддерживающем интеграцию чата с выдачей Яндекса

На данный момент доступно 5 сервисов, которые поддерживают интеграцию чата с поисковой выдачей, это:

- JivoSite
- Talk-me
- Verbox
- Bitrix
- LiveTex

Рассмотрим интеграцию на примере сервиса Talk-me

Регистрируем у них бесплатный аккаунт и после этого в админке добавляем свой сайт

![](/assets/img/posts/2018/08/17/pic-2018-08-17_14-51-29.png){: w="300" }
_добавить сайт_

Переходим во вкладку `Интеграция и API` и слева в списке выбираем `Яндекс.Диалоги`

![](/assets/img/posts/2018/08/17/pic-2018-08-17_14-53-38.png){: w="300" }
_Яндекс.Диалоги_

Нажимаем кнопку `Добавить`, после чего будет `сгенерирован ID`. Он нам понадобится на следующем этапе.

## Регистрация в сервисе Яндекс.Диалоги

Нам нужно перейти в сервис Яндекс.Диалоги и нажать `Вход для разработчиков`.

Далее нажимаем `Создать диалог`.


![](/assets/img/posts/2018/08/17/pic-2018-08-17_14-41-29.png){: w="300" }
_Создать диалог_

Выбрать тип диалога `Чат для бизнеса`

![](/assets/img/posts/2018/08/17/pic-2018-08-17_14-42-45.png){: w="300" }
_Чат для бизнеса_

Заполняем:

Основные настройки:
- Сайт для верификации прав использования бренда: `адрес вашего сайта` (сайт должен быть верифицирован в Яндекс.Вебмастер)
- Провайдер: `Talk-Me`
- ID: Сюда вписываем `ID`, который мы получили у провайдера Talk-Me
- URL: не обязательно

Публикация в каталоге:
- Название компании: я написал название сайта
- Название диалога: так же написал название сайта
- Категория: выбрать подходящую категорию
- Приветственное сообщение: любое приветствие
- Саджесты: готовые вопросы пользователей (см. примечание в Яндекс.Диалоги)
- Иконка: иконка 192×192 пикселя
- Рабочее время: выбрать подходящее

Жмем сохранить и отправляем на модерацию.

Через некоторое время в результатах поиска будет отображаться кнопка "Чат с компанией"
