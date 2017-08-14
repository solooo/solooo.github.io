---
title: maven3自定义archetype
date: 2017-08-14 19:13:28
categories: 
- maven

tags:
- maven
---
maven使用起来还是很方便，但默认自带的archetype配置junit版本比较老。每次创建新项目都要手动修改junit版本，所以就想着能不能自己建一个新版本出来，省得每次手动修改的麻烦。
网上找了下教程，发现还是很简单的。maven提供了一种非常快速的创建骨架模板的方式，那就是**create-from-project**,可以让你直接使用当前项目创建archetype。下面记录一下创建过程，以普通的springboot工程为例，创建自己的archetype:
<!-- more -->

#### 创建普通springboot工程，项目名：Demo
创建一个基本的springboot工程，添加依赖，集成swagger2自动生成api文档。springboot+mybatis为主要框架，使用mybatis-generator生成sql映射文件。项目配置不细说，搭建完成后添加示例代码测试OK即可。

#### 创建archetype
在*Demo*项目根目录(`{Demo-root}`)下执行命令 `mvn archetype:create-from-project`。注意此时项目`{Demo-root}/target/generated-sources/`目录下会生成**archetype**文件夹。到此步可以说已经创建了一个archetype,只是没有安装到仓库，暂时还无法使用

#### 安装到本地仓库
命令行`cd`进入`{Demo-root}/target/generated-sources/archetype`目录下，此处需要修改以下两处地方：

【A】删除不需要的多余文件
修改`src/main/resources/META-INF/maven/archetype-metadata.xml`文件。其中节点`fileSet`定义一个文件夹，子节点`directory`定义文件夹位置，把不需要构建到模板文件夹删除即可。
例：![fileSet](maven3自定义archetype/fileSet.png)

【B】增加私有仓库部署路径，需要让其他人可以使用此模板，必须增加这个配置
在当前目录下的`pom.xml`文件中添加以下配置项
```xml
<distributionManagement>
    <repository>
      <id>nexus-releases</id>
      <name>nexus Releases</name>
      <url>http://{ip}:{port}/nexus/content/repositories/releases/</url>
    </repository>
    <snapshotRepository>
      <id>nexus-snapshots</id>
      <name>nexus Snapshots</name>
      <url>http://{ip}:{port}/nexus/content/repositories/snapshots/</url>
    </snapshotRepository>
</distributionManagement>
```
其中节点`<id>`需要指定为maven配置文件setting.xml所配置的角色ID，两处ID需要对应。例如：
```xml
<!--配置权限,使用默认用户-->
<servers>
    <server>
        <id>nexus-releases</id>
        <username>deployment</username>
        <password>deployment123</password>
    </server>
    <server> 
        <id>nexus-snapshots</id>
        <username>deployment</username>
        <password>deployment123</password>
    </server>
</servers>
```

#### 安装部署
在第三步的目录下(`target/generated-sources/archetype`),执行以下两个命令：
    
`mvn install` 只能安装到本地仓库，其他人无法使用。
    
`mvn deploy` 发布到私服，其他人也可以使用

#### 完成
项目模板生成后pom.xml(`{Demo-root}/target/generated-sources/archetype/pom.xml`)中会对`artifactId`添加后辍`-archetype`。可以记录此文件的gav直接添加使用模板，或者从Nexus中查找模板坐标使用
自定义archetype完成，可以愉快使用了！