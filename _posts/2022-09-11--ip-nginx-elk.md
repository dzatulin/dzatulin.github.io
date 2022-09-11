---
title: "Рейтинг ip адресов по количеству запросов Nginx в ELK"
date: 2022-09-11
categories:
  - Blog
tags:
  - ELK
---
По умолчанию стандартный дашборд Nginx в ELK не формирует рейтинг ip по количеству запросов.
Для формирования отдельной визуализации необходим Fields: nginx.access.remote_ip_list. Данная директива передается заголовком X-Forwarded-For, должна передаваться в Nginx logs.

Настройки визуализации: 
<img src="https://dzatulin.github.io/assets/images/elk_nginx.jpg">

Пример визуализации: 
<img src="https://dzatulin.github.io/assets/images/elk_nginx2.jpg">