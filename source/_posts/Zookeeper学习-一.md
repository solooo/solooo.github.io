---
title: Zookeeper学习(一)
date: 2017-05-12 22:29:16
categories:
- technology
tags:
- zookeeper
- java
---
### Zookeeper介绍
>ZooKeeper is a centralized service for maintaining configuration information, naming, providing distributed synchronization, and providing group services. All of these kinds of services are used in some form or another by distributed applications. Each time they are implemented there is a lot of work that goes into fixing the bugs and race conditions that are inevitable. Because of the difficulty of implementing these kinds of services, applications initially usually skimp on them ,which make them brittle in the presence of change and difficult to manage. Even when done correctly, different implementations of these services lead to management complexity when the applications are deployed.

以上是Zookeeper官网说明，Zookeeper主要提供的服务有：配置管理、名称服务、分布式同步、分组服务。
<!-- more -->
### 使用场景
以下是从网上一篇博客引用来的。[阅读原文](http://www.cnblogs.com/yuyijq/p/3424473.html)
>##### 配置管理
>
>在我们的应用中除了代码外，还有一些就是各种配置。比如数据库连接等。一般我们都是使用配置文件的方式，在代码中引入这些配置文件。但是当我们只有一种配置，只有一台服务器，并且不经常修改的时候，使用配置文件是一个很好的做法，但是如果我们配置非常多，有很多服务器都需要这个配置，而且还可能是动态的话使用配置文件就不是个好主意了。这个时候往往需要寻找一种集中管理配置的方法，我们在这个集中的地方修改了配置，所有对这个配置感兴趣的都可以获得变更。比如我们可以把配置放在数据库里，然后所有需要配置的服务都去这个数据库读取配置。但是，因为很多服务的正常运行都非常依赖这个配置，所以需要这个集中提供配置服务的服务具备很高的可靠性。一般我们可以用一个集群来提供这个配置服务，但是用集群提升可靠性，那如何保证配置在集群中的一致性呢？ 这个时候就需要使用一种实现了一致性协议的服务了。Zookeeper就是这种服务，它使用Zab这种一致性协议来提供一致性。现在有很多开源项目使用Zookeeper来维护配置，比如在HBase中，客户端就是连接一个Zookeeper，获得必要的HBase集群的配置信息，然后才可以进一步操作。还有在开源的消息队列Kafka中，也使用Zookeeper来维护broker的信息。在Alibaba开源的SOA框架Dubbo中也广泛的使用Zookeeper管理一些配置来实现服务治理。
>
>##### 名字服务
>
>名字服务这个就很好理解了。比如为了通过网络访问一个系统，我们得知道对方的IP地址，但是IP地址对人非常不友好，这个时候我们就需要使用域名来访问。但是计算机是不能是别域名的。怎么办呢？如果我们每台机器里都备有一份域名到IP地址的映射，这个倒是能解决一部分问题，但是如果域名对应的IP发生变化了又该怎么办呢？于是我们有了DNS这个东西。我们只需要访问一个大家熟知的(known)的点，它就会告诉你这个域名对应的IP是什么。在我们的应用中也会存在很多这类问题，特别是在我们的服务特别多的时候，如果我们在本地保存服务的地址的时候将非常不方便，但是如果我们只需要访问一个大家都熟知的访问点，这里提供统一的入口，那么维护起来将方便得多了。
>
>##### 分布式锁
>
>其实在第一篇文章中已经介绍了Zookeeper是一个分布式协调服务。这样我们就可以利用Zookeeper来协调多个分布式进程之间的活动。比如在一个分布式环境中，为了提高可靠性，我们的集群的每台服务器上都部署着同样的服务。但是，一件事情如果集群中的每个服务器都进行的话，那相互之间就要协调，编程起来将非常复杂。而如果我们只让一个服务进行操作，那又存在单点。通常还有一种做法就是使用分布式锁，在某个时刻只让一个服务去干活，当这台服务出问题的时候锁释放，立即fail over到另外的服务。这在很多分布式系统中都是这么做，这种设计有一个更好听的名字叫Leader Election(leader选举)。比如HBase的Master就是采用这种机制。但要注意的是分布式锁跟同一个进程的锁还是有区别的，所以使用的时候要比同一个进程里的锁更谨慎的使用。
>
>##### 集群管理
>
>在分布式的集群中，经常会由于各种原因，比如硬件故障，软件故障，网络问题，有些节点会进进出出。有新的节点加入进来，也有老的节点退出集群。这个时候，集群中其他机器需要感知到这种变化，然后根据这种变化做出对应的决策。比如我们是一个分布式存储系统，有一个中央控制节点负责存储的分配，当有新的存储进来的时候我们要根据现在集群目前的状态来分配存储节点。这个时候我们就需要动态感知到集群目前的状态。还有，比如一个分布式的SOA架构中，服务是一个集群提供的，当消费者访问某个服务时，就需要采用某种机制发现现在有哪些节点可以提供该服务(这也称之为服务发现，比如Alibaba开源的SOA框架Dubbo就采用了Zookeeper作为服务发现的底层机制)。还有开源的Kafka队列就采用了Zookeeper作为Cosnumer的上下线管理。

### 安装

安装过程非常简单

1.下载文件，此次下载为最新发行版：zookeeper-3.4.10.tar.gz
2.解压文件，复制并重命名文件conf/zoo_simple.cfg为zoo.cfg
3.修改文件内容：
    
    # 心跳间隔时间
    tickTime=2000
    # 数据目录
    dataDir=D:\\zookeeper
    # 服务端口
    clientPort=2181
    
4.进入bin目录，双击zkServer.cmd启动服务即可

java示例，直接使用Zookeeper API操作：
```java
public class ZookeeperTest {

    public static void main(String[] args) {
        String content = "this is test测试";
        try {
            ZooKeeper zooKeeper = new ZooKeeper("127.0.0.1:2181", 3000, null);
            // 注册
            zooKeeper.register(new ZkWatcher(zooKeeper, "/root"));
            // 创建节点
            zooKeeper.create("/root", "root data".getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
            // 设置数据
            zooKeeper.setData("/root", content.getBytes(), -1);
            Stat stat = new Stat();
            System.out.println(new String(zooKeeper.getData("/root", false, stat)));

            zooKeeper.create("/root/child", "child".getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
            // 节点删除
            zooKeeper.delete("/root/child", -1);
            zooKeeper.delete("/root", -1);

        } catch (IOException | InterruptedException | KeeperException e) {
            e.printStackTrace();
        }
    }

    public static class ZkWatcher implements Watcher {

        private ZooKeeper zooKeeper;
        private String path;

        public ZkWatcher(ZooKeeper zooKeeper, String path) {
            this.zooKeeper = zooKeeper;
            this.path = path;
        }

        @Override
        public void process(WatchedEvent watchedEvent) {
            System.out.println("Watcher: " + watchedEvent.getType());
        }
    }
}
```


使用ZkClient操作Zookeeper,此包封装了对zk的一些常用操作

Maven包引用：
```xml
<dependency>
    <groupId>com.101tec</groupId>
    <artifactId>zkclient</artifactId>
    <version>0.10</version>
</dependency>
```

```java
public class ZkClientTest {

    public static final String PATH = "/zkClient";

    public static void main(String[] args) {
        ZkClient client = new ZkClient("127.0.0.1:2181");
        client.subscribeDataChanges(PATH, new IZkDataListener() {
            @Override
            public void handleDataChange(String s, Object o) throws Exception {
                System.out.println("handleDataChange s: " + s + " data: " + o);
            }

            @Override
            public void handleDataDeleted(String s) throws Exception {
                System.out.println("handleDataDeleted s:" + s);
            }
        });

        client.create(PATH, "school", CreateMode.PERSISTENT);
        client.writeData(PATH, "城中");
        System.out.println(client.readData(PATH));

        client.create(PATH + "/child", "child", CreateMode.PERSISTENT);
        System.out.println(client.getChildren(PATH));
        client.writeData(PATH + "/child", "高三三班");
        System.out.println(client.readData(PATH + "/child"));

        client.delete(PATH + "/child");
        client.delete(PATH);
    }
}

```
以上是对Zookeeper的基本操作，具体的使用还在学习中。

参考博客：http://www.cnblogs.com/powercto/p/6844798.html