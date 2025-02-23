---
title: "Filter and Remove Docker Images, Containers"
date: 2023-01-08
categories:
  - Blog
tags:
  - System
---

For quick and selective removal of Docker containers and images, standard commands are not always sufficient:

- **`docker system prune`**: Removes all unused resources not associated with containers.
- **`docker system prune -a`**: Removes all stopped containers and unused resources.
- **`docker images purge`**: Removes images not tied to containers.
- **`docker rmi $(docker images -a -q)`**: Removes all images.
- **`docker rm $(docker ps -a -f status=exited -q)`**: Removes all stopped containers.
- **`docker volume prune`**: Removes unused volumes.

For more selective image or container removal, you can use combinations of `grep` and `awk` for fine-grained control.

### Removing Images

```bash
docker images -a | grep "repo/app:1.0" | awk '{print $3}' | xargs docker rmi
```
