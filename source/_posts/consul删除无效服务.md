---
title: consul删除无效服务
date: 2019-01-28 15:31:53
categories: linux
tags: consul
---
微服务注册中心 consul 删除无效服务需要通过 api 发送请求，UI 页面未提供入口

```bash
curl -X PUT http://localhost:8500/v1/agent/service/deregister/microservice-gateway-8000
```
其中 `microservice-gateway-8000` 为服务名称

删除无效节点
```bash
curl -X PUT http://127.0.0.1:8500/v1/agent/force-leave/{节点id}
```
