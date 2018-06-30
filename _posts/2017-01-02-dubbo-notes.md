---
layout: article
title: Dubbo 2.5.3 学习笔记
tags: Dubbo
key: 2017-01-02-dubbo-notes
---
# Dubbo 2.5.3 学习笔记

## Dubbo 介绍
Dubbo (http://dubbo.io/) 是阿里巴巴公司开源的一个高性能优秀的服务框架，使得应用可通过高性能的 RPC 实现服务的输出和输入功能，可以和 Spring框架无缝集成。

本文写于 17 年初，当时的 Dubbo 基本已经不再更新一段时间，版本是 2.5.3，17 年底的时候捐献给了 Apache 孵化项目，又开始更新。新的官网 (http://dubbo.apache.org/)

架构图：
![](http://dubbo.apache.org/images/dubbo-architecture.png)
 
节点角色说明：
- Provider: 暴露服务的服务提供方。
-	Consumer: 调用远程服务的服务消费方。
-	Registry: 服务注册与发现的注册中心。
-	Monitor: 统计服务的调用次调和调用时间的监控中心。
-	Container: 服务运行容器。

项目 URL：
-	源码：https://github.com/alibaba/dubbo 新地址：https://github.com/apache/incubator-dubbo
-	下载：http://repo1.maven.org/maven2/com/alibaba/dubbo
-	http://repo1.maven.org/maven2/com/alibaba/dubbo/2.5.3/ 
-	微博：http://weibo.com/dubbo

当前版本 2.5.3, 23 Oct 2012

## Zookeeper

### Zookeeper 的介绍
Zookeeper (http://zookeeper.apache.org/) 一般作为 Dubbo 的注册中心，当前版本 3.4.9

文档：https://zookeeper.apache.org/doc/current/zookeeperStarted.html

### 运行 Zookeeper
下载  (3.4.9 老地址已失效，最新地址：https://mirrors.tuna.tsinghua.edu.cn/apache/zookeeper/zookeeper-3.4.12/zookeeper-3.4.12.tar.gz)
```
$ wget https://mirrors.tuna.tsinghua.edu.cn/apache/zookeeper/zookeeper-3.4.9/zookeeper-3.4.9.tar.gz
$ tar -zxvf zookeeper-3.4.9.tar.gz
```

创建配置文件 `zoo.cfg`
```
$ cp conf/zoo_sample.cfg conf/zoo.cfg
```

设置 dataDir 和 dataLogDir :
```
dataDir=/home/yin/Dubbo/zookeeper-3.4.9/data
dataLogDir=/home/yin/Dubbo/zookeeper-3.4.9/logs
```

其他参数如：
```
tickTime=2000
dataDir=/var/lib/zookeeper
clientPort=2181
```

启动:
```
$ cd ~/Dubbo/zookeeper-3.4.9/bin/
$ ./zkServer.sh start
```

检查状态：
```
$ ./zkServer.sh status
```

停止服务
```
$ ./zkServer.sh stop
```

查看进程
```
$ jps
```
有个进程：QuorumPeerMain


### zkCli 连接到 ZooKeeper 服务
```
$ bin/zkCli.sh -server 127.0.0.1:2181
```

### 运行复制的 ZooKeeper
Zoo.cfg
```
tickTime=2000
dataDir=/var/lib/zookeeper
clientPort=2181
initLimit=5
syncLimit=2
server.1=zoo1:2888:3888
server.2=zoo2:2888:3888
server.3=zoo3:2888:3888
```

在文件 data/myid 里设置 id，文件内容就一个 id：
```
1
```

### Zkclient
有多种 Zookeeper 的客户端程序

#### com.github.adyliu	zkclient	2.1.1
支持 zk 3.4.5, 最后更新日期 2013 年
https://github.com/adyliu/zkclient
```xml
<dependency>
    <groupId>com.github.adyliu</groupId>
    <artifactId>zkclient</artifactId>
    <version>2.1.1</version>
</dependency>
```

#### com.101tec	zkclient	0.10
https://github.com/sgroschupf/zkclient
支持 zookeeper 3.4.8, Dubbo 官方推荐的
最后更新: 2016
```xml
<dependency>
    <groupId>com.101tec</groupId>
    <artifactId>zkclient</artifactId>
    <version>0.10</version>
</dependency>
```

### com.github.sgroschupf	zkclient	0.1
(last update: 2011, same author as 101tec, subbort zk 3.3.3)
https://github.com/sgroschupf/zkclient
```xml
<dependency>
    <groupId>com.github.sgroschupf</groupId>
    <artifactId>zkclient</artifactId>
    <version>0.1</version>
</dependency>
```
### Apache Curator
https://github.com/Netflix/curator
http://curator.apache.org/
http://blog.csdn.net/lzy_lizhiyang/article/details/48518731

## Dubbo Admin

老版本下载：dubbo-admin-2.5.3.war，新版本在这里：https://github.com/apache/incubator-dubbo-ops

Using Tomcat:
`$ unzip dubbo-admin-2.5.3.war -d ROOT`

Edit WEB-INF/dubbo.properties
```
dubbo.registry.address=zookeeper://10.229.15.96:2181
dubbo.admin.root.password=root
dubbo.admin.guest.password=guest
```

> 注意当前版本的 Dubbo Admin 只能使用 jdk1.7 (不能使用 1.8). See: http://www.cnblogs.com/digdeep/p/5205537.html

在 Tomcat 的 catalina.bat and setclasspath.bat 里设置：
```
set JAVA_HOME=D:\Tools\jdk1.7.0_80
set JRE_HOME=D:\Tools\jdk1.7.0_80\jre
```

设置 host，启动
```
$ cd ~/Dubbo/dubbo-admin/bin
$ ./startup.sh
```
访问 http://devops.wilmar.cn:8787 用账号 root/root 登入

(小 Bug：如果发布到非 ROOT 上下文，比如： http://localhost:8787/dubbo-admin-2.5.3 ，有些连接会失效，建议发布到 ROOT，或者修复连接：WEB-INF\templates\sysinfo\layout\default.vm 文件里，修改
`onclick="window.location.href='/sysinfo/$tab';"`
为
`onclick="window.location.href='$rootContextPath.getURI("/sysinfo/$tab")';"`
)

### Dubbo Monitor
Dubbo 的监控程序，当前版本 2.5.3
```
$ tar -zxvf dubbo-monitor-simple-2.5.3-assembly.tar.gz
```

编辑 /home/yin/Dubbo/dubbo-monitor-simple-2.5.3/conf/dubbo.properties
```
dubbo.container=log4j,spring,registry,jetty
dubbo.application.name=simple-monitor
dubbo.application.owner=
#dubbo.registry.address=multicast://224.5.6.7:1234
dubbo.registry.address=zookeeper://127.0.0.1:2181
#dubbo.registry.address=redis://127.0.0.1:6379
#dubbo.registry.address=dubbo://127.0.0.1:9090
dubbo.protocol.port=7070
dubbo.jetty.port=8686
dubbo.jetty.directory=${user.home}/Dubbo/dubbo-monitor-simple-2.5.3
dubbo.charts.directory=${dubbo.jetty.directory}/charts
dubbo.statistics.directory=${user.home}/monitor/statistics
dubbo.log4j.file=logs/dubbo-monitor-simple.log
dubbo.log4j.level=WARN
```
Eg: dubbo.registry.address=zookeeper://10.135.108.152:2181?backup=10.135.108.153:2181,10.135.108.154:2181
```
$ cd ~/Dubbo/dubbo-monitor-simple-2.5.3/bin/
$ ./start.sh
```
http://devops.wilmar.cn:8686/

Set in app:
```xml
<!-- monitor -->
<dubbo:monitor protocol="registry" />
```

### Provider App
Maven 依赖
```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>dubbo</artifactId>
    <version>2.5.3</version>
    <exclusions>
        <exclusion>
            <groupId>org.springframework</groupId>
            <artifactId>spring</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```
zookeeper:
```xml
<dependency>
    <groupId>org.apache.zookeeper</groupId>
    <artifactId>zookeeper</artifactId>
    <version>3.4.9</version>
    <exclusions>
        <exclusion>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
        </exclusion>
        <exclusion>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
        </exclusion>
    </exclusions>
</dependency>

<dependency>
    <groupId>com.github.sgroschupf</groupId>
    <artifactId>zkclient</artifactId>
    <version>0.1</version>
</dependency>
```

spring-dubbo.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://code.alibabatech.com/schema/dubbo
       http://code.alibabatech.com/schema/dubbo/dubbo.xsd">

    <!-- 提供方应用信息，用于计算依赖关系 -->
    <dubbo:application name="dubbo-provider"/>

    <!-- 使用multicast广播注册中心暴露服务地址 -->
<!--    <dubbo:registry address="multicast://224.5.6.7:1234"/>-->
    <dubbo:registry address="zookeeper://10.229.15.96:2181" check="false" subscribe="false" register=""/>
    <!-- 用dubbo协议在20880端口暴露服务 -->
    <dubbo:protocol name="dubbo" port="20880"/>

    <!-- 声明需要暴露的服务接口 -->
    <dubbo:service interface="cn.wilmar.api.HelloService" ref="helloService"/>

</beans>
```

App
```java
@SpringBootApplication
@ImportResource({"classpath:spring-dubbo.xml"})
public class HelloProviderApplication {
```

Some service:
```java
@Service("helloService")
public class HelloServiceImpl implements HelloService {
```

### Consumer App
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://code.alibabatech.com/schema/dubbo
       http://code.alibabatech.com/schema/dubbo/dubbo.xsd">

    <!-- 消费方应用名，用于计算依赖关系，不是匹配条件，不要与提供方一样 -->
    <dubbo:application name="dubbo-consumer"  />

    <!-- 使用multicast广播注册中心暴露发现服务地址 -->
<!--
    <dubbo:registry address="multicast://224.5.6.7:1234" />
-->
    <dubbo:registry address="zookeeper://10.229.15.96:2181"/>
    <!--can't add:  check="false" subscribe="false"
    register=""-->
    <!-- 生成远程服务代理，可以和本地bean一样使用demoService -->
    <dubbo:reference id="helloService" interface="cn.wilmar.api.HelloService" />

</beans>
```

Close check at startup:
```xml
<!-- 生成远程服务代理，可以和本地bean一样使用demoService -->
<dubbo:reference id="helloService" interface="cn.wilmar.api.HelloService" check="false" />
```

注入 Service:
```java
@Autowired
HelloService helloService;
```

Cache
```java
<dubbo:reference id="helloService" interface="cn.wilmar.api.HelloService" check="false" cache="true" />
```

Cluster:
http://blog.csdn.net/zuoanyinxiang/article/details/50977072

```xml
<!-- 声明需要暴露的服务接口 -->
<dubbo:service interface="cn.wilmar.api.HelloService" ref="helloService" cluster="failover"/>
```

Loadbalance
```xml
<!-- 声明需要暴露的服务接口 -->
<dubbo:service interface="cn.wilmar.api.HelloService" ref="helloService" cluster="failover" loadbalance="random"/>
```

Multicast Register
http://blog.csdn.net/zuoanyinxiang/article/details/50980106

```xml
<dubbo:registry address="multicast://224.5.6.7:1234"/>
<dubbo:registry address="zookeeper://10.229.15.96:2181" check="false" subscribe="false"
                register="" default="false"/>
```

只订阅（不注册）
适用开发环境开发
`<dubbo:registry server=”” register=”false”`

在reference里面（订阅）加上url指定dubbo服务
（但是不适合代码提交，可以用jvm参数）

只注册（不订阅）
`Subscribe=”false”`

其他关于Dubbo的学习，必看的资料：
-	用户指南
-	开发指南
-	管理指南
-	常见问题
其实就是首页http://dubbo.io/ 下面的四个链接，里面内容编排比较乱，都看就是了
在下载栏目里的大部分下载地址都失效了，我东拼西凑找了差不多，放在：
http://pan.baidu.com/s/1kVfTj2f

国内还吴水成架构师做的培训视频：http://pan.baidu.com/s/1i5DhkmL
