---
title: Docker-Images
date: 2022-01-08 14:30:08
tags: Docker
cover: /images/Moby-logo.png
---

# Docker-Images 制作镜像

Images镜像是一个描述了应用运行所需环境/依赖的只读文件。

所以构建镜像需要描述应用的运行环境/依赖，这个描述文件是Dockerfile。

有了Docker Images，我们可以面向Images编程，无需关心操作系统的差异性。

## Dockerfile

- FROM，基础镜像
- ADD，复制代码 or 其他文件

- RUN，构建镜像时运行
- CMD，容器运行时运行

**例子**

```Dockerfile
FROM node:11 # 基础镜像

MAINTAINER serialrick.top # 维护者

# 把宿主机的.复制到/app（通常用于复制代码）
ADD . /app

# 容器运行后进入此目录
WORKDIR /app

# 构建镜像时需要运行的命令，可以有多个
RUN npm install

# 运行容器时执行的命令，只能有一个
CMD node app.js
```

## build images

**制作镜像命令**

指定了Dockerfile在.路径

`docker build -t myimage .`

## 镜像仓库

https://hub.docker.com/

# Reference

[制作自己的镜像](https://docker.easydoc.net/doc/81170005/cCewZWoN/N9VtYIIi)

[Dockerfile](https://docs.docker.com/engine/reference/builder/#run)

