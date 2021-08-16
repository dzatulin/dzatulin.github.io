---
title: "Monitoring Elasticsearch in Kibana"
date: 2020-11-04
categories:
  - Blog
tags:
  - Elasticsearch
  - Kibana
---
  В Kibana есть хороший детализированый мониторинг кластера Elasticsearch. Для его использования необходимо установить модуль X-PAC. 
  
**X-Pack**  представляет собой плагин к Elasticsearch и Kibana. 

**Stack Monitoring** можно разделить на два основных направления:
- Мониторинг производительности Elasticsearch.
- Мониторинг Kibana и маршрутизации данных в кластер.

  <img src="https://dzatulin.github.io/assets/images/stack_mon.jpg">

В случии использования **BASIC** лицензии **X-PAC** мы можем мониторить один кластер Elasticsearch. Базовая настройка использует локальный кластер в нашем случае кластер Elasticsearch вынесен на удаеленный сервер.  

Для этого:

- Включаем сбор данных в кластере Elasticsearch.

```yml
  xpack.monitoring.collection.enabled: true
```

- Добавляем модуль metricbeat

```bash
  metricbeat modules enable elasticsearch-xpack
```

- Настраиваем модуль 

```yml
  - module: elasticsearch
    xpack.enabled: true
    period: 10s
    hosts: ["http://my.host.io:9200"] 
    #scope: node 
    #username: "user"
    #password: "secret"
    #ssl.enabled: true
    #ssl.certificate_authorities: ["/etc/pki/root/ca.pem"]
    #ssl.certificate: "/etc/pki/client/cert.pem"
    #ssl.key: "/etc/pki/client/cert.key"
    #ssl.verification_mode: "full"
    xpack.enabled: true
```

- Настраиваем в Kibana место расположения данных

 По умолчанию, кластер определяется параметром `elasticsearch.hosts`, в нашем случае сервер удаленный определен параметром 

```yml
 monitoring.ui.elasticsearch.host  my.host.io
```

- Запускаем серверную часть мониторинга в Kibana

```yml
  monitoring.ui.enabled true
```

- Если включен модуль безопасности необходимо указать логин, пароль  и добавить пользователя в `monitoring_user` рорль 

```yml
 monitoring.ui.elasticsearch.username user
 monitoring.ui.elasticsearch.password pwd 
```

- Откройте **Stack Monitoring** в Kibana 
