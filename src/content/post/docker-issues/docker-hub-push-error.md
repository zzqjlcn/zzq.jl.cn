---
title: "Docker 镜像拉取问题"
description: "DockerHub和其它国外的镜像仓库被封，如何万无一失的同步和下载国外的镜像到国内？"
publishDate: "2025-07-07"
seriesId: docker-issue
orderInSeries: 1
tags: ["docker", "dockerhub", "image"]
---

# Docker 镜像下载问题
> 继03年`Docker Hub`被不知名原因封锁后，导致各大NAS的镜像库或注册表都无法使用，不能直接呈现列表。不过很久大家就发现通过docker pull或者其他的镜像源依然可以下载镜像，所以虽说注册表无法访问，但也无伤大雅。而目前，不管是通过docker pull还是通过修改镜像源，都无法访问docker镜像库了，这就导致所有存于docker hub的镜像均无法下载，极空间也不例外。

# 采取国内私有云拉取

> 目前国内的镜像仓库有很多，最常用的有 DaoCloud、阿里云、腾讯云等。
## 1. DaoCloud 镜像仓库
目前测试可行的方案：https://docs.daocloud.io/community/mirror.html

可以使用脚本拉取 `docker-pull.sh`
```shell
#!/bin/bash

img=$1

docker pull m.daocloud.io/docker.io/$img \ 
    && docker tag m.daocloud.io/docker.io/$img $img \
    && docker rmi m.daocloud.io/docker.io/$img
```
拉取python镜像：
```shell
docker-pull.sh python:3.10-slim-buster
```