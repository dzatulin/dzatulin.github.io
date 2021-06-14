---
title: "Nginx rewrite and redirect rules"
date: 2020-08-16
categories:
  - Blog
tags: 
  - Nginx
---

Редирект - важный элемент оптимизации сайта, мощный инструмент для управления трафиком. В этой статье я собрал полезные команды, примеры rewrite и redirect с которыми мне приходилось сталкиваться в работе. Отдельно выделяю **ngx_http_rewrite_module**, модуль изменения URI запроса с помощью регулярных выражений. 

Модуль **ngx_http_rewrite_module** позволяет изменять URI запроса с помощью регулярных выражений PCRE, делать перенаправления и выбирать конфигурацию по условию.
Директивы **break**, **if**, **return**, **rewrite** и **set** последовательно выполняютс, описанные на уровне server.

Директивы в цикле:
{% capture notice-2 %}
* ищется location по URI запроса;
* последовательно выполняются директивы этого модуля, описанные в найденном location;
* цикл повторяется, если URI запроса изменялся, но не более 10 раз.
{% endcapture %}

<div class="notice">{{ notice-2 | markdownify }}</div>

**Return** завершает обработку и возвращает клиенту указанный код(301, 302, 303, 307 и 308).

```
return код URL;
```

**Rewrite** можно использовать для перезаписи URL в NGINX. Как и return директива **rewrite** может быть помещена в блок server, а так же в блок location. **Rewrite** измененият входящий URL в новый URL, извлекает элементы из входящего URL-адреса, которые не имеют переменных в Nginx, что делает rewrite более функциональным чем **return**.  

```
rewrite regex замена [флаг];
```

- **regex** регулярное выражение на основе PCRE, которое будет использоваться для сопоставления с URI входящего запроса;


- **замена** если регулярное выражение совпадает с запрошенным URI, тогда строка замены используется для изменения запрошенного URI;

- **флаг** значение флага определяет, нужна ли дополнительная обработка директивы rewrite или нет.

**Rewrite** Может возвращать только коды 301 или 302, что бы вернуть другие коды, необходимо добавить директиву return после rewrite.

{% capture notice-2 %}
### Флаги:

* **last**  - завершает обработку текущего набора директив модуля ngx_http_rewrite_module, после чего ищется новый location, соответствующий изменённому URI;
* **break** - завершает обработку текущего набора директив модуля ngx_http_rewrite_module;
* **redirect** - возвращает временное перенаправление с кодом 302; используется, если заменяющая строка не начинается с “http://”, “https://” или “$scheme”;
* **permanent** - возвращает постоянное перенаправление с кодом 301.
{% endcapture %}

<div class="notice">{{ notice-2 | markdownify }}</div>

### Шпаргалка по регулярным выражениям 

- . -  точка в регулярном выражении обозначает «любой символ, кроме переноса строки».
- ^ -  представляет начало строки для сопоставления.
- $ -  представляет конец строки для сопоставления.
- ? -  повторять символ 0 или 1 раз, тоже самое, что и {0,1}. Например, «colou?r» соответствует и color, и colour.
- ```*``` -  повторять символ от 0 до 65536 раз, означает 0, 1 или любое число раз ({0,})
- ```+``` -  плюс означает хотя бы 1 раз ({1,}). Например, «go+gle» соответствует gogle, google и т. д. (но не ggle)
- ```\.``` - Так как точка специальный символ, то для того, чтобы обозначить точку, ее нужно экранировать слешем.
- \w - любой символ, который может составить слово
- \w+ - любое количество таких символов (один или больше).

### Примеры Rewrite и Redirect

- Редирект с www на non-www

```
server {
  listen 80;
  server_name www.example.org;
  return 301 $scheme://example.org$request_uri;

server {
  listen 80;
  server_name example.org;
}
}
```

- Редирект с HTTP на HTTPS 

```
server {
  listen 80;
  return 301 https://$host$request_uri;
}

server {
  listen 443 ssl;

  # let the browsers know that we only accept HTTPS
  add_header Strict-Transport-Security max-age=2592000;
}
```

- Добавляет слэш / в конце каждого URL.

a) с помощью rewrite, только в том случаее если в URL нет точки или параметров.

```
rewrite ^([^.\?]*[^/])$ $1/ permanent;
```

b) с помощью return:

```
if ($request_uri ~ "^(.*)[^/]$") {
    return 301 $1/;
}
```

- Удаляем слэш / в конце строки 

a) с помощью rewrite:

```
rewrite ^/(.*)/$ /$1 permanent;
```

b) с помощью return:

```
if ($request_uri ~ "^(.*)/$") {
   return 301 /$1;
}
```

- Редирект на страницу

a) с помощью rewrite:

```
rewrite ^/page1$ /page2 permanent;
```

b) с помощью return:

```
location = /page1.html {
  return 301 http://example.org/page2.html;
}
```

- Редирект на сайт

```
server {
  server_name site1.com
  return 301 $scheme://site2.com$request_uri;
}
```
 
- Удалить часть URL 

a) с помощью rewrite:

```
location /path1 {
  rewrite ^/path1/(.*) http://example.org/path2/$1 permanent;
}
```

b) с помощью оператора if

```
if ( $request_filename ~ path1/.+ ) {
     rewrite ^(.*) http://www.site.com/path2/$1 permanent;
}

if ($request_uri ~ "/deleted-url/(.*)") {
     return 301 $1;
}
```

- Если в URL есть аргумент lang и его значение равно en, то редиректить нужно на en.example.com, сохранив URL без аргументов.

```
server {
    listen 80;
    server_name example.com;
    location / {
        if ($arg_lang = en) { return 302 http://en.example.com$uri ; }
    }
}
```

- Cкрывать index.php в конце строки и в url

```
if ($request_uri ~* "^(.*/)index\.php/*(.*)") {
    return 301 $1$2;
}
```

