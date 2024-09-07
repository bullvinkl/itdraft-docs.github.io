---
title: "Подключаем Elasticsearch к Wordpress, настройка плагина ElasticPress (Autosuggest) для Nginx"
date: "2023-07-20"
categories: 
  - Content-Management
tags: 
  - docker
  - docker-compose
  - elasticpress
  - elasticsearch
  - linux
  - nginx
  - wordpress
image:
  path: /commons/pexels-lukas-574071-scaled.jpg
  alt: "Подключаем Elasticsearch к Wordpress, настройка плагина ElasticPress (Autosuggest) для Nginx"
---

> Elasticsearch – быстрое, распределенное, масштабируемое решение с открытым кодом, предназначенное для управления поисковым контентом. Elasticsearch можно легко масштабировать за счет его распределенной архитектуры.
{: .prompt-tip }

Устанавливаем Elasticsearch на сервер. На момент написания статьи ElasticPress совместим с Elasticsearch не выше версии 7.10. 

Если ваша система с Wordpress в Docker исполнении, часть конфига из docker-composer:
```sh
$ nano docker-composer.yml
...
  elasticsearch:
    image: elasticsearch:7.10.1
    container_name: elasticsearch
    restart: always
#    ports:
#      - "9200:9200"
    environment:
      - discovery.type=single-node
      - bootstrap.memory_lock=true
      - xpack.security.enabled=false
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - ./elasticsearch:/usr/share/elasticsearch/data
    logging:
        driver: "json-file"
        options:
            max-size: "10k"
            max-file: "10"
...
```

Редактируем конфиг Nginx
```sh
$ nano ./nginx/wp.conf
...
location /ep-autosuggest {
  # only allow POST requests
  limit_except POST {
    deny all;
  }

  # Perform our request
  rewrite ^/ep-autosuggest(.*) $1/_search break;
  proxy_set_header Host $host;

  # Use the URL of the server here
  proxy_pass http://elasticsearch:9200;
  return 403;
}
...
```

Где строчка `proxy_pass http://elasticsearch:9200;` - для Wordpress в docker исполнении.

Перезагружаем Nginx через docker composer
```sh
$ docker composer restart nginx
```

Устанавливаем плагин ElasticPress, активируем его, производим первичную настройку и синхронизацию
![](/assets/img/posts/2024/07/20/image-5.png){: w="300" }
_Активируем ElasticPress_

Переходим в пункт меню Features, подключаем поиск Elasticsearch в разделе Post Search
![](/assets/img/posts/2024/07/20/image-6.png){: w="300" }
_Подключаем поиск Elasticsearch_

Раскрываем раздел Autosuggest и прописываем настройки:
- Autosuggest Selector: `input[type="search"]`
- Endpoint URL: `https://itdraft.ru/ep-autosuggest` - добавляли в Nginx

![](/assets/img/posts/2024/07/20/image-7.png){: w="300" }
_Прописываем настройки в Autosuggest_

Пример работы:
![](/assets/img/posts/2024/07/20/image-9-1024x394.png){: w="300" }
_Пример работы_
