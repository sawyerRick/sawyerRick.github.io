---
title: Docker-Command
date: 2022-01-08 14:14:59
tags: Docker
cover: /images/Moby-logo.png
---

# Docker-Command

总结我常用的Docker命令

- 进入容器的shell：docker exec -it containerId /bin/bash
- 查看所有正在运行的容器：docker ps
- 查看所有容器：docker ps -a



有着挂载、网络、stdin交互功能地运行容器

`docker run -it -p 8080:8080 --mount type=bind,src="$(pwd)",target=/app --network=test-net webapp`

