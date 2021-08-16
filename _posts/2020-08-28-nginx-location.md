---
title: "Nginx location"
date: 2020-08-28
categories:
  - Blog
tags:
  - Nginx
---

Директива **location** устанавливает конфигурацию в зависимости от URI запроса. Позволяет направить запрос на нужное нам местоположение в файловой системе. Имеет следующий синтаксис:

```
location [модификатор] [uri] {
       . . .
}
```

Модификатор в блоке не обязателен, наличие его в блоке позволяет по-разному обрабатывать URL. Несколько наиболее распространённых модификаторов.

{% capture notice-2 %}
### Модификаторы


* отсутствует  –  если модификатор отсутствует, указанное местоположение будет сопоставлено с началом URI запроса для определения совпадения;
* =    знак равенства предназначен для строго соответствия между запрошенным URI и значением location, если условие совпадает то поиск прекращается;
* ~    (тильда) регистрозависимое соотношение означает, что это местоположение будет интерпретировано как совпадение регулярного выражения;
* ~*    регистро не зависимое соотношение означает, что это местоположение будет интерпретировано как совпадение регулярного выражения;
* ^~   (карет,тильда) исключает поиск регулярных выражений.
{% endcapture %}

<div class="notice">{{ notice-2 | markdownify }}</div>

- 1 Базовый location

```
location / {
    # Совпадает с URI всех запросов, т.к. они все начинаются с "/"
    # Но! Если будут найдены соответствия в расположениях
    # с регулярными выражениями или с другим более длинным
    # строковым литералом (например, "/data/"),
    # то конфигурация для "/" не будет применена.
}
```

- 2 Корневой  URI «/»

```
location = / {
    # Расположение только для URI /
    # При совпадении дальнейший поиск не осуществляется
}
```

- 3 URI вида «/data/.*»

```
location /data/ {
    # Это расположение соответствует всем URI, начинающихся с "/data/"
    # и продолжает поиск по оставшимся location'ам.
    # В этом примере регулярные выражения и другие строковые литералы
    # также будут проверены, и "location /data/" будет использован,
    # если более конкретизирующие расположения не удовлетворят искомому ресурсу.
}
```

- 4 URI вида «/img/.*»

```
location ^~ /img/ {
    # Соответствует любому URI, начинающемуся с "/img/".
    # Если соответствие установлено - останавливает дальнейший поиск,
    # не проверяя регулярные выражения, т.к. используется
    # префикс ^~ "крышечка/циркумфлекс тильда".
}
```

- 5 Для графических форматов

```
location ~* \.(png|ico|gif|jpg|jpeg)$ {
    # Данное расположение соответствует всем запрашиваемым URI,
    # оканчивающихся ".png", ".ico", ".gif", ".jpg" или ".jpeg".
    # При этом надо заметить, что запросы внутри расположения "/img/"
    # будут обработаны в location из примера №4
    # Т.е., если расположения из примеров №4 и №5 разместить в таком же
    # порядке в реальном конфиге, то до этого расположения ресурсов 
    # дойдут только те картинки, которые лежат вне расположения "/img/"
}
```

- 6  Именованный location

```
location / {
    error_page 404 = @fallback;
}

location @fallback {
    # Если при внутреннем перенаправлении не нужно менять URI,
    # то можно передать обработку ошибки в именованный location
}
```

Примеры некоторых **location**:

- Защита от Hotlink

Директива location для Anti-hotlinking (борьбы с использованием ресурсов с вашего сервера на сторонних ресурсах. Такой способ использования ваших сетевых ресурсов называется hotlinking). Такое поведение хитрых разработчиков может заметно увеличить нагрузку на ваш сервер. Конфигурация:

```
location ~ \.(gif|png|jpe?g)$ {
    valid_referers none blocked mywebsite.com *.mywebsite.com;
    if ($invalid_referer) {
       return 403;
    }
}
```

- Запрет на скрипты внутри директорий

Следующий пример — запрет на скрипты в разрешённых для записи директориях:

```
location ~* /(images|cache|media|logs|tmp)/.*\.(php|pl|py|jsp|asp|sh|cgi)$ {
    error_page 403 /403.htm;
    return 403;
}
```

- Открыт доступ к медиа файлам

```
location ^~ /uploads/.tmb {
      location ~ \.(png|ico|gif|jpg|jpeg)$ {
      }
      return 403;
}
```