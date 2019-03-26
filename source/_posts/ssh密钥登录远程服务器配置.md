---
title: ssh密钥登录远程服务器配置
date: 2019-01-28 14:04:25
categories: Linux
tags: ssh
---

配置无密码登录远程服务器，本机 `ubuntu18.04`, 服务器 `centos`

首选确保远程服务器安装配置`ssh`服务，此处使用阿里云，默认安装此服务。以下所有命令均在本地执行

#### 1、生成 ssh 公钥密钥文件
命令：
```bash
$ ssh-keygen
```
<!-- more -->
根据提示输入文件名，或者使用默认文件名`id_rsa`，一路回车即可，待返回如下内容表示创建成功
```bash
Generating public/private rsa key pair.
Enter file in which to save the key (/home/eric/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/eric/.ssh/id_rsa.
Your public key has been saved in /home/eric/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:F74RLcqlTC9Le9kzY9XjHppuVhjouDxRTESvzcF9ZWE eric@eric
The key's randomart image is:
+---[RSA 2048]----+
|           .++oE=|
|         . +  .o.|
|        . O +   .|
|       + * O . . |
|        S * = +..|
|       . = O +...|
|        o + O .o |
|         . = *+ .|
|          . =+ . |
+----[SHA256]-----+
```
查看目录`~/.ssh`，生成如下两个文件（默认文件名）:
```bash
id_rsa
id_rsa.pub
```
#### 2、上传公钥文件到服务器
命令:
```bash
ssh-copy-id username@hostname
```
其中 username 为远程服务器登录名， hostname 为远程服务器 IP 或域名

#### 3、登录服务器
经过以上配置后，本地即可免密码登录服务器

命令：
```bash
ssh username@hostname
```
#### 4、配置本地 ssh 记住连接地址
每次输入用户名和 IP 比较麻烦，可在 Terminal 中配置好用户名和地址，使用别名登录，具体配置如下：

编辑（没有就新建一个）`~/.ssh/config`:
```
Host alias_name             # 服务器别名
HostName xx.xx.xxx.xxx      # 服务器IP或域名
User test                   # 服务器登录用户名
Port 22                     # ssh 登录端口
IdentityFile ~/.ssh/id_rsa  # 密钥文件，不需要后辍
```
保存后，可在 Terminal 中使用以下命令直接登录服务器：
```bash
ssh alias_name
```
若有多个服务器，在 config 文件中继续添加一条配置即可，格式和内容与上一条相同
