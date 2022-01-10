---
title: Docker-Network
date: 2022-01-08 14:43:11
tags: Docker
cover: /images/Moby-logo.png
---

# Docker-Network

构建一个Docker网络，不同容器之间就可以通过某个网络互相访问。

## Command

创建一个网络

`docker network create my-network`

使Redis容器运行在网络中（并给自己IP取别名）

`docker run -d --network --network-alias redis redis:latest`

使web项目运行在网络中（代码中可以通过alias当成ip访问）

`docker run -it -p 8080:8080 --network=test-net webapp`

