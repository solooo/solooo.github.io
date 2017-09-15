---
title: Docker学习(二)-Dockerfile
date: 2017-09-15 17:52:56
categories:
- technology
tags:
- docker
---
### 介绍
很多情况下我们需要自己制作镜像，打包自己的应用放到docker上执行。Dockerfile就是打包镜像的利器

制作Docker镜像有两种方式：

* 一、启动一个基础窗口，在其中进行定制化操作后，commit当前容器，则会将之前一系列操作保存到镜像；
* 二、使用Dockerfile创建镜像

第一种不建议使用，一般用来被入侵后保存现场等场景中。主要创建方式是通过Dockerfile方式

Dockerfile 是一个文本文件，其内包含了一条条的指令(Instruction)，每一条指令构建一层，因此每一条指令的内容，就是描述该层应当如何构建。
<!-- more -->
### 示例

以定制JDK1.8镜像为例

首先，创建空目录`myimg`，在新目录中执行`touch Dockerfile`来创建空的Dockerfile文件，然后复制官网下载的jdk1.8安装包(`jdk-8u144-linux-x64.tar.gz`)到*myimg*中

编辑Docker内容如下：
```
FROM centos:7
ADD jdk-8u144-linux-x64.tar.gz /usr/java/
ENV JRE_HOME=/usr/java/jdk1.8.0_144
```
逐行解释：

第1行：`FROM`指令指定镜像依赖的基础镜像，这里我用的官方的centos7镜像做为基础镜像。所有操作都在此镜像基础上进行

第2行：`ADD`指令作用为添加文件到镜像中，如果为压缩包，会自动解压；可以直接使用http链接，打包镜像时会自动下载文件

第3行：`EVN`指令作用为设置环境变量

在*myimg*目录中执行以下命令
```bash
docker build -t myimg .
```
将看到制作过程，结束后可用`docker images`查看镜像列表，是否存在自制镜像

### Dockerfile指令详解

##### FROM
格式
```
FROM  <image>
or
FROM <image>:<tag>
```
必须做为第一行，指定镜像基准。

##### COPY
格式
```
COPY <源路径>... <目标路径>
or
COPY ["<源路径1>",... "<目标路径>"]
```
COPY 指令将从构建上下文目录中 <源路径> 的文件/目录复制到新的一层的镜像内的 <目标路径> 位置。例如：
```
COPY start.sh /opt/
```
<源路径> 可以是多个，甚至可以是通配符；

<目标路径> 可以是容器内的绝对路径，也可以是相对于工作目录的相对路径（工作目录可以用 WORKDIR 指令来指定）。目标路径不需要事先创建，如果目录不存在会在复制文件前先行创建缺失目录。
                     
此外，还需要注意一点，使用 COPY 指令，源文件的各种元数据都会保留。比如读、写、执行权限、文件变更时间等。这个特性对于镜像定制很有用。特别是构建相关文件都在使用 Git 进行管理的时候。

##### ADD
格式与COPY一样，基本作用也相似，但功能比COPY强大

如果源路径是压缩包，上传后会自动解压。

源路径支持超链接，可从远程下载文件。但如果链接下载的是压缩包，*不会自动解压*，所以，不建议使用

此指令只建议需要上传本地压缩包并解压的时候使用，其他情况还是使用COPY比较好

##### RUN
格式
```
RUN <命令/可执行文件> # shell格式
or
RUN ["可执行文件", "参数1", "参数2"] #exec格式
```
在当前镜像的顶层执行任何命令，并commit成新的镜像.

RUN格式有两种写法。区别如下：

shell格式，相当于执行`/bin/sh -c "<command>"`：

exec格式，不会触发shell，所以$HOME这样的环境变量无法使用，但它可以在没有bash的镜像中执行，而且可以避免错误的解析命令字符串：

Dockerfile每条指令都会创建新的层，所以当有多条命令需要执行时，尽量合并在一个RUN指令里。多条执行命令：
```
RUN buildDeps='gcc libc6-dev make' \
    && apt-get update \
    && apt-get install -y $buildDeps \
    && wget -O redis.tar.gz "http://download.redis.io/releases/redis-3.2.5.tar.gz" \
    && mkdir -p /usr/src/redis \
    && tar -xzf redis.tar.gz -C /usr/src/redis --strip-components=1 \
    && make -C /usr/src/redis \
    && make -C /usr/src/redis install \
    && rm -rf /var/lib/apt/lists/* \
    && rm redis.tar.gz \
    && rm -r /usr/src/redis \
    && apt-get purge -y --auto-remove $buildDeps
```

##### CMD
格式与**RUN**相同，也区分shell和exec两种格式

用于指定默认的容器主进程的启动命令的

在指令格式上，一般推荐使用 exec 格式，这类格式在解析时会被解析为 JSON 数组，因此一定要使用双引号 "，而不要使用单引号

>容器中应用在前台执行和后台执行的问题:
>
>Docker 不是虚拟机，容器中的应用都应该以前台执行，而不是像虚拟机、物理机里面那样，用 upstart/systemd 去启动后台服务，容器内没有后台服务的概念。
>
>一些初学者将 CMD 写为：CMD service nginx start
 然后发现容器执行后就立即退出了。甚至在容器内去使用 systemctl 命令结果却发现根本执行不了。这就是因为没有搞明白前台、后台的概念，没有区分容器和虚拟机的差异，依旧在以传统虚拟机的角度去理解容器。
>
>对于容器而言，其启动程序就是容器应用进程，容器就是为了主进程而存在的，主进程退出，容器就失去了存在的意义，从而退出，其它辅助进程不是它需要关心的东西。

##### ENV
格式
```
ENV <key> <value>
ENV <key1>=<value1> <key2>=<value2>...
```
作用是设置环境变量。无论是后面的其它指令，如 RUN，还是运行时的应用，都可以直接使用这里定义的环境变量
