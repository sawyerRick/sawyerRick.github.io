---
title: Docker-Mount
date: 2022-01-07 22:20:02
tags: Docker
cover: /images/Moby-logo.png
---

# Docker Mount

Docker 提供三种挂载技术：

1. bind mount
2. volume，
3. tmpfs

使用`--mount`或`-v`参数进行mount，`--mount`和`-v`唯一不同的地方在于：

- 如果src不存在，`-v`会自动创建，而`--mount`则不会自动创建并抛出错误

## 1. Bind mount

bind mount通常用于本地更新代码



### Example

以下命令把宿主机的pwd/target挂载到容器的/app目录

```shell
docker run -d \
  -it \
  --name devtest \
  --mount type=bind,source="$(pwd)"/target,target=/app \
  nginx:latest
```



## 2. Volume（recommend）

Docker Volume是一种持久化和数据共享技术，由Docker在host宿主机管理，可以把Volume 挂载（mount）到容器的文件夹上实现持久化功能。

1. 有了Volume，容器停止后文件数据也可以保存。
2. 不同容器可以共享同一个Volume

#### Commands

--mount和-v命令相同作用，但在service命令中只能用--mount。

1. docker volume ls，查看所有volume
2. docker volume create [volume-name]，创建volume
3. docker inspect volume-name，查看volume
4. --mount source=myvol2,target=/app，挂载volume到某个容器文件路径

#### Example

创建一个Volume myvol，挂载到容器的/app上

```shell
docker run -d \
  --name devtest0 \
  --mount source=myvol,target=/app \
  nginx:latest
```

验证Volume持久化和共享功能：

进入devtest0的/app，创建一些文件

新开一个容器devtest1，执行相同命令，发现/app仍然有数据，验证了Volume的持久化功能。



