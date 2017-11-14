---
title: Docker  openjdk-8-jdk-alpine 容器时间与jdk时区不同修改方法
date: 2017-11-14 13:42:36
categories:
- technology
tags:
- docker
---

测试时发现以 openjdk-8-jdk-alpine 为基础镜像制作的镜像有时区问题。查资料知道 alpine 本身不带时区信息。按以下方法修改后正常获取时间。
####  一、挂载宿主机的时区到容器
启动命令增加参数  `-v /etc/localtime:/etc/localtime`

启动容器，进入查看时间正常，时区也已同步。但java应用获取的时间还是差8小时。

继续查找资料 java 获取时区与 linux 系统时区的不同 [参考博客](https://my.oschina.net/huawu/blog/4646)

#### 二、设置容器内时区
通过后来不断尝试，修改 localtime 确定无效，最后采用笨方法，制作应用镜像时，通过 shell 修改时区

Dockerfile 中增加修改时区的命令 `echo "Asia/Shanghai" > /etc/timezone`

通过测试，jdk 中或正常获取设置的东8时区。但还有一个遗留问题，使用 springboot 启动项目，打印出来的时间时间还是默认时区，而应用中获取的时间都是正常的时间。暂且记下，有时间再细看