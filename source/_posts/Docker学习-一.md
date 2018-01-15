---
title: Docker学习(一)
date: 2017-09-15 11:49:04
categories:
- technology
tags:
- docker
---
docker学习笔记

#### 安装Docker
安装参考阿里云国内镜像安装，比用官方国外速度快很多。

阿里云提供自动安装脚本可以很方便安装docker,shell 中执行以下命令：
    
    curl -sSL http://acs-public-mirror.oss-cn-hangzhou.aliyuncs.com/docker-engine/internet | sh -

参考地址：[Docker CE 镜像源站](https://yq.aliyun.com/articles/110806)

文档写的很清楚，具体不再详述
<!-- more -->
#### Docker国内镜像加速器
国内访问Docker Hub速度很慢，推荐使用国内的镜像加速器。国内提供镜像加速器的服务有很多，我用的[阿里云加速器](https://cr.console.aliyun.com/#/accelerator)

>如何使用Docker加速器
>
>针对Docker客户端版本大于1.10的用户
>
>您可以通过修改daemon配置文件`/etc/docker/daemon.json`来使用加速器：
```bash
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://x216c2zd.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

#### 常用命令
镜像、容器
```bash
## 显示docker镜像
docker images
## 删除docker镜像
docker rmi -f [imageid]
## 停止docker容器
docker stop [containerId]
## 删除docker容器
docker rm -f [containerId]

## 显示docker容器
docker ps
    -a 		列出所有容器,含已停止运行容器
    -f 		[exited=<int>] 列出满足 exited=<int> 条件的容器
    -l 		仅列出最新创建的一个容器
    -q 		仅列出容器ID
    -s 		显示容器大小
    -n=4 	列出最近创建的4个容器
    --no-trunc  显示完整的容器ID
    --before="nginx"  列出在某一容器之前创建的容器, 接受容器名称和ID作为参数
    --since="nginx"   列出在某一容器之后创建的容器, 接受容器名称和ID作为参数
```

启动：
```bash
docker run [imageId]
    -it    : -i 表示交互; -t 表示终端; 合起来就是交互式终端
    -d     : 后台运行
    -p     : 默认桥接网络模式, 映射端口
    -v     : 挂载容器和主机间的路径
    --rm   : 容器停止后删除容器
    --net=host : 网络主机模式
    --restart=always : 随着docker服务开机启动
    --link={containerName}:myhost : 在容器中建立另一个容器containerName连接,myhost代替连接地址或IP
```
上传下载
```bash
##下载
docker pull [ip]:[port]/tomcat:8
##上传
##打版本
docker tag tomcat:8 [ip]:[port]/tomcat:8
##上传
docker push [ip]:[port]/tomcat:8
```

其它命令
```bash
## 进入容器
docker exec [containerId] -it bash

## 执行命令
docker exec [containerId] -it [command]

## 容器控制台日志
docker logs -f [containerId]

###清理###
#杀死所有正在运行的容器
docker kill $(docker ps -a -q)

#删除所有已经停止的容器
docker rm $(docker ps -a -q)

#删除所有未打 dangling 标签的镜像
docker rmi $(docker images -q -f dangling=true)

#删除所有镜像
docker rmi $(docker images -q)
```