---
title: "Filter and Remove Docker Images, Containers"
date: 2023-01-08
categories:
  - Blog
tags:
  - System
---

Для быстрого и выборочного удаления docker containers и images не всегда достаточно стандартных команд: 

**docker system prune** - удаление всех ресурсов, несвязанных с контейнерами.
**docker system prune -a**  удаление всех остановленных контейнеров и ресурсов.
**docker images purge** - удаление образов не привязанных к контейнерам. 
**docker rmi $(docker images -a -q)** - удаление всех образов.
**docker rm $(docker ps -a -f status=exited -q)** - удаление всех контейнеров.
**docker volume prune** - удаление не связанных томов.

Для более выборочного удаления образов или контейнеров можно использовать удобные комбинации grep и awk:

Images:

```
docker images -a | grep "repo/app:1.0" | awk '{print $3}' | xargs docker rmi
```

```
docker rmi $(docker images "repo/app:1.0" -q | tail -n 50) 
```

Containers: 

```
docker ps -a | grep "repo/app:1.0" | awk '{print $3}' | xargs docker rm
```

```
docker ps -a -f ancestor="repo/app:1.0" -f status="exited" | grep 'days ago' | awk '{print $1}' | xargs --no-run-if-empty docker rm   
```