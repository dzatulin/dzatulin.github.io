---
title: "Logstash+Gelf Docker logs"
date: 2022-01-29
categories:
  - Blog
tags:
  - Monitoring
  - ELK
---
**GELF**(The Graylog Extended Log Format) - это формат лога, который помогает избегать недостатки стандартного Syslog:
 - увеличивает стандартную длину сообщения syslog 1024 байта;  
 - доступна прямая интеграция с приложениями(мы используем Symphony+Gelf);
 - отсутствие типов сообщений как Syslog "Facilities";
 - Каждое сообщение лога в GELF содержит поля: (host, timestamp, version, message, custom fields).

 GELF разработан для **Graylog** - мощная платформа с открытым исходным кодом для управления структурированными и неструктурированными данными, а также для отладки приложений. Graylog в сочетании с MongoDB и Elasticsearch часто сравнивают с ELK (Elasticsearch, Logstash и Kibana). Оба решения дают возможность решать похожие задачи(сбор, хранение, анализ логов).

 Пример лога Gelf в JSON:
 ```
 {
  "version": "1.1",
  "host": "example.org",
  "short_message": "A short message that helps you identify what is going on",
  "full_message": "Backtrace here\n\nmore stuff",
  "timestamp": 1385053862.3072,
  "level": 1,
  "_user_id": 9001,
  "_some_info": "foo",
  "_some_env_var": "bar"
}
 ```
 Для того, что бы не беспокоиться о тайм-аутах, проблемах с подключением, которое может сломать ваше приложение, GELF может использовать UDP.

 Пример UDP Input Gelf для Logstah:
 ```
 input {
   gelf {
      use_udp => true
      port_udp => 12201
   }
 }
 ```
### Docker поддержка Gelf 

Docker поддерживает gelf log driver, что дает возможность удобно мониторить приложение Docker с помощью Graylog, Logstash, Fluentd. 

Контейнеры, логи которых мы хотим видеть в ELK или Graylog, должны иметь конфигурацию лога:
```
logging:
  driver: gelf
  options:
    gelf-address: udp://logstash.server:12201
```
Если вы запускаете docker вместо docker-compose:

```
docker run --log-driver gelf --log-opt gelf-address=udp://logstash.server:12201
```
Для удобного форматирования можно добавить tag:

```
logging:
  options:
    tag: staging
```
Пример лога: 
```
{
  "_index": "containers-stage-2022.01.30",
  "_type": "_doc",
  "_id": "veAKq34BrClNybG_pVzt",
  "_score": 1,
  "_source": {
    "level": 3,
    "source_host": "57.190.13.124",
    "type": "dockerstage",
    "message": "127.0.0.1 -  30/Jan/2022:14:51:48 +0200 \"GET /index.php\" 400",
    "host": "storage-px62-fsn1",
    "@version": "1",
    "version": "1.1",
    "image_name": "containers/stage-service:stage",
    "@timestamp": "2022-01-30T12:51:48.742Z",
    "container_id": "90bda0ee69a266ff0d066340dd4265b3a04aafca9514178d76e7baf5f6a67952",
    "tag": "stage-lms-service",
    "container_name": "service-02",
    "image_id": "sha256:f7da54deee7649be2a1e6b89a8695d394adc5c34650de775891ff845a4b596a9",
    "created": "2022-01-28T20:54:10.727435635Z",
    "command": "/usr/bin/supervisord -n -c /etc/supervisord.conf"
  }
```