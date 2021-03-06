---
title: Docker Registry https 部署
date: 2018-07-05 12:44:04
categories:
- technology
tags:
- docker
---
现在 `Docker Registry` 默认都启用 `https` 访问，虽然可以通过配置其他节点的 `daemon.json` 文件来使用 `http` 访问，但如此操作比较麻烦，且限制节点仓库地址。
在部署 `Kubernetes` 时，必须要有一个可以方便访问的仓库，于是参考网络资料，尝试搭建带证书的私有仓库并记录过程如下
<!-- more -->
#### openssl生成密钥文件
1. 修改`openssl`配置文件，编辑`/etc/pki/tls/openssl.cnf`,在[ v3_ca ]下增加了一行：
```
[ v3_ca ]
subjectAltName=IP:192.168.1.6
```
`192.168.1.6`就是 `Registry` 所在的宿主机 IP
这个很重要，否则在后面会报registry endpoint...x509: cannot validate certificate for ... because it doesn't contain any IP SANs

2. 生成密钥
```
openssl req -newkey rsa:4096 -nodes -sha256 -keyout certs/domain.key -x509 -days 365 -out certs/domain.crt
```
在证书的创建过程中，会询问国家、省分、城市、组织、部门和common name的信息，其中`common name`信息填写主机的IP 192.168.1.6

证书创建完毕后，在certs目录下出现了两个文件:
```sh
domain.key # 证书文件
domain.crt # 私钥文件
```
#### 启动仓库加入证书
3. 仓库启动命令
```
docker run -d \
 --restart=always \
 --name registry \
 -v /root/certs:/root/certs \
 -v /var/lib/registry:/var/lib/registry \
 -e REGISTRY_HTTP_ADDR=0.0.0.0:5000 \
 -e REGISTRY_HTTP_TLS_CERTIFICATE=/root/certs/domain.cert \
 -e REGISTRY_HTTP_TLS_KEY=/root/certs/domain.key \
 -p 5000:5000 \
 registry
 ```
 可在浏览器访问：`https://192.168.1.6:5000/v2` 测试仓库是否可用。正常会返回`{}`

#### 其他主机使用仓库
4. 将第三步生成的密钥文件复制到需要使用仓库的主机上

在其他主机上创建目录`mkdir -p /etc/docker/certs.d/192.168.1.6:5000`

目录中 IP 端口即仓库地址
```
scp certs/domain.crt root@ht240:/etc/docker/certs.d/192.168.1.6:5000/ca.crt
```
无需重启`docker`服务即可使用私有仓库

#### 仓库Web界面 Doceker Registry Web
`docker-registry-web`一个简单的仓库显示UI，只能显示不能做其他任何操作
```
docker run -d -p 9001:8080 --name registry-web \
           -e REGISTRY_URL=https://192.168.1.6:5000/v2 \
           -e REGISTRY_TRUST_ANY_SSL=true \
           -e REGISTRY_NAME=192.168.1.6:5000 \
		   hyper/docker-registry-web
```