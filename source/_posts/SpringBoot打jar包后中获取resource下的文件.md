---
title: SpringBoot打jar包后中获取resource下的文件
date: 2019-03-26 09:29:49
categories: code
tags: code
---
jar 包中获取resource下的文件
```java
String context = null;
InputStream resourceAsStream = Thread.currentThread().getContextClassLoader().getResourceAsStream("applicationEx.json");
if (resourceAsStream != null) {
    context = IOUtils.toString(resourceAsStream);
}
```