---
title: SpringBoot 上传文件临时目录失效问题解决
date: 2018-08-21 14:48:28
categories:
tags:
---
SpringBoot 上传文件时会创建临时目录暂存文件，但临时目录会被系统自动清理掉导致上传失败，解决办法之一就是修改默认的临时文件目录：

SpringBoot 执行文件增加以下代码，自定义文件目录
```java
/**
 * 文件上传临时目录修改
 * @return
 */
@Bean
MultipartConfigElement multipartConfigElement() {
    MultipartConfigFactory factory = new MultipartConfigFactory();
    String tmpPath = rootPath + File.separator + "tmp";
    Path path = Paths.get(tmpPath);
    if (!Files.exists(path) || !Files.isDirectory(path)) {
        try {
            Files.createDirectories(path);
        } catch (IOException e) {
            logger.error("tmp 目录创建失败! 请手动创建此目录：{}", tmpPath);
        }
    }
    factory.setLocation(tmpPath);
    return factory.createMultipartConfig();
}
```