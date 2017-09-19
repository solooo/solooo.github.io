---
title: Linux JDK 环境变量
date: 2017-02-06 11:06:50
categories:
- technology
tags:
- Linux
- JDK
---

## 系统版本

Ubuntu16.04.1

### 编辑文件

> /etc/profile

<!-- more -->
### 文件末尾添加

```shell
#set jdk environment
export JAVA_HOME=/usr/lib/jvm/jdk1.7.0_21
export CLASSPATH=.:$JAVA_HOME/lib:$JAVA_HOME/jre/lib:$CLASSPATH
export PATH=$JAVA_HOME/bin:$JAVA_HOME/jre/bin:$PATH
```

### 使文件生效

> source /etc/profile

OK!
