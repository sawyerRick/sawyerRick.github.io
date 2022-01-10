---
title: Docker-Compose
date: 2022-01-08 15:31:28
tags: Docker
cover: /images/Moby-logo.png
---

# Docker-Compose

> Compose is a tool for defining and running multi-container Docker applications.

Docker Compose可以组合多个容器，使多容器应用可以做到一键构建运行（COOL ! ）。

使用`docker-compose.yml`文件描述多容器应用。

## docker-compose.yml

一个简单的包括一个webapp和redis的例子，其中有用到不同的挂载方式(bind mount&volume)和不同的启动方式（build/images)。

> Notes: 根据Docker Compose启动的应用，内部所有应用默认都在同一个虚拟网络中

```yml
version: "3.7"

services:
  app: # 应用1
    build: ./ # 在此目录构建运行
    ports:
      - 80:8080
    volumes: # 使用bind mount
      - ./:/app
    environment:
      - TZ=Asia/Shanghai
  redis: # 应用2
    image: redis:5.0.13 # 使用镜像运行
    volumes: # 使用volume
      - redis:/data
    environment:
      - TZ=Asia/Shanghai

volumes:
  redis:
```

## Command

**启动**

`docker compose up`

# Reference

[Docker](https://docs.docker.com/compose/)

[Docker教程](https://docker.easydoc.net/doc/81170005/cCewZWoN/IJJcUk5J)
