---
title: "Remove node from Elasticsearch custer"
date: 2021-06-15
categories:
  - Blog
tags:
  - ELK
---
Для того, что бы извлечь ноду из кластера Elasticsearch, необходимо выполнить команду.
```
curl -XPUT localhost:9200/_cluster/settings -H 'Content-Type: application/json' -d '{
  "transient" :{
      "cluster.routing.allocation.exclude._ip" : "10.0.0.1"
   }
}';echo
```

После того как шарды будут распределены между остальными нодами, можно выполнять отключение/профилктику/замену сервера. 

Для возвращения сервера в кластер необходимо выполнить запрос с пустым значением ip:

```
curl -XPUT localhost:9200/_cluster/settings -H 'Content-Type: application/json' -d '{
  "transient" :{
      "cluster.routing.allocation.exclude._ip" : ""
   }
}';echo
```
