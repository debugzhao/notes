### 1 入门

#### 1.1 概述

Zookeeper是一个开源的分布式的，管理分布式服务的Apache项目

Zookeeper从设计模式的角度理解，是一个基于**观察者模式**的分布式服务管理框架，它负责**存储和管理集群的数据（集群元数据信息：ip地址、服务器上下线状态）**，并且接受观察者的注册。一旦这些数据发生变化，zookeeper就负责**通知**已经注册的那些观察者，并做出相应的反应

**Zookeeper事件通知流程**

<img src="https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/image.42yz95m5coc0.png" alt="image" style="zoom:50%;" />

1. 服务端启动时向zookeeper注册信息
2. 客户端可以获取到当前在线的服务器列表，并且注册监控服务器状态
3. 如果有一个服务器下线
4. zookeeper将会发送服务器节点下线的通知事件
5. 客户端收到注册中心的通知后，会重新获取当前在线的服务器列表，并注册监听

综合以上的zookeeper事件通知流程，可以任务**zookeeper = 文件系统 + 事件通知系统**

#### 1.2 特点

1. zookeeper架构

   <img src="https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/image.515abz6kb3o0.png" alt="image" style="zoom:50%;" />

2. zookeeper特点

   1. zookeeper集群中只有一个leader，可以有多个follower

      写数据操作只能往leader里面写，读数据可以从多个follower读

   2. 集群中只要有**半数以上**的服务器存活，zookeeper集群就能正常服务，所有zookeeper适合安装**奇数台**服务器

   3. 全局数据一直性

      每个节点中都持久化了一份相同的数据副本，client无论连接到哪个server，数据都是一致的

   4. 更新请求顺序执行

      来自同一个客户端的更新请求，按照其发送的顺序依次执行

   5. 数据更新原子性

      一次数据更新要么成功，要么失败

   6. 实时性

      在一定的时间范围内，client可以读到其他客户端更新的最新数据

#### 1.3 数据结构

ZooKeeper 数据模型的结构与 Unix 文件系统很类似，整体上可以看作是**一棵树**，每个 节点称做一个 ZNode。每一个 ZNode 默认能够**存储 1MB** 的数据，每个 ZNode 都可以通过 其**路径作为唯一标识**

#### 1.4 应用场景

1. 统一命名服务

   在分布式环境下，经常需要对应用/服 务进行统一命名，便于识别

   例如：IP不容易记住，而域名容易记住

   <img src="https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/image.49f24e5oozg0.png" alt="image" style="zoom: 50%;" />

2. 统一配置管理

   分布式环境下，配置文件同步非常常见。一般要求一个集群中，所有节点的配置信息是 一致的，对配置文件修改后，希望能够快速同步到各个 节点上。

   可将配置信息写入ZooKeeper上的一个Znode，可将配置信息写入ZooKeeper上的一个Znode，一 旦Znode中的数据被修改，ZooKeeper将通知 各个服务器同步最新的配置文件

   <img src="https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/image.3zqlewlnje80.png" alt="image" style="zoom:50%;" />

3. 统一集群管理

   分布式环境中，实时掌握每个节点的状态是必要的，ZooKeeper可以实现实时监控节点状态变化。可将节点信息写入ZooKeeper上的一个ZNode。监听这个ZNode可获取它的实时状态变化

4. 节点动态上下线

   <img src="https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/image.at1paxwaia0.png" alt="image" style="zoom: 50%;" />

5. 软负载均衡

   在Zookeeper中记录每台服务器的访问数，让访问数最少的服务器去处理最新的客户端请求

   <img src="https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/image.6ipswp27q2g0.png" alt="image" style="zoom: 50%;" />

### 2 本地安装

#### 2.1 本地模式安装

#### 2.2 配置参数解读

```yaml
  1 # The number of milliseconds of each tick
  2 tickTime=2000
  3 # The number of ticks that the initial
  4 # synchronization phase can take
  5 initLimit=10
  6 # The number of ticks that can pass between
  7 # sending a request and getting an acknowledgement
  8 syncLimit=5
  9 # the directory where the snapshot is stored.
 10 # do not use /tmp for storage, /tmp here is just
 11 # example sakes.
 12 dataDir=/usr/local/zookeeper-3.5.7/data
 13 # the port at which the clients will connect
 14 clientPort=2181
```

1. tickTime=2000

   zookeeper客户端和服务器通信心跳时间，单位毫秒

   <img src="C:\Users\lucas.zhao\AppData\Roaming\Typora\typora-user-images\image-20210725094237278.png" alt="image-20210725094237278" style="zoom:67%;" />

2. initLimit=10

   leader 和follower初始建立连接时最多可以容忍的心跳数，10次 * 2000ms = 20s 也就是第一次通信时超过20s 还没有建立连接，leader则认为follow挂掉

3. syncLimit=5

   leader 和followe同步次数限制，超过 5次 * 2000ms = 10s 连接失败，leader则认为follower挂掉

4. dataDir

   zookeeper保存数据的目录

5. clientPort

   端口号

### 3 集群操作

#### 3.1 集群操作

##### 集群安装

**zoo.cfg配置文件说明**

```yaml
#################cluster#####################
server.3=172.20.18.163:2888:3888
server.4=172.20.18.164:2888:3888
server.5=172.20.18.165:2888:3888
```

**server.A=B:C:D 配置参数解读**

1. A 是一个数字，表示这个是第几号服务器

   集群模式下配置一个文件 myid，这个文件在 dataDir 目录下，这个文件里面有一个数据 就是 A 的值，Zookeeper 启动时读取此文件，拿到里面的数据与 zoo.cfg 里面的配置信息比 较从而判断到底是哪个 server

2. B 表示节点ip地址

3. C是这个服务器 Follower 与集群中的 Leader 服务器交换信息的端口

4. D 是万一集群中的 Leader 服务器挂了，需要一个端口来重新进行选举，选出一个新的 Leader，而这个端口就是用来执行选举时服务器相互通信的端口

##### ❗选举机制（面试重点）

假设集群中一共五个服务器

> 1. SID
>
>    服务器id，用来唯一标识zookeeper集群中的节点，和myid保持一致
>
> 2. ZXID
>
>    事务id，用来标识每一次服务器状态的变更
>
> 3. Epoch
>
>    每个leader的任期代号。没有leader时同一轮投票过程中的逻辑时钟值是相同的，每投完一次票这个数值就会增加

1. 第一次启动时的选举机制

   <img src="https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/image.1bkeiyl3ek80.png" alt="image" style="zoom:50%;" />

   1. 阶段1：server1启动

      server 1启动发起一次选举，首先server1 **投自己一票**。此时server1的的票数没有超过半数以上（3票），选举无法完成，server1 的状态保持为`LOOKING状态`

   2. 阶段2：server2启动

      server2 启动，再次发起一次选举。server1和serve2分别**投递自己一票**并且交换自己的选票信息（**myid**）。此时server1 发现server2的myid比自己的大，于是将自己的选票投递给server2。此时server1票数为0票，server2票为2票，没有超过半数3票，选举失败。server1，server2分别保持`LOOKING状态`

   3. 阶段3：server3启动

      server3启动，再次发起一次选举。此时server1和server2分别将自己的票数投递给server3（**server3的myid最大**），投票结束。此时投票结果为：server1票数0票，server2票数0票，server3票数3票。此时server3的票数已经超过半数以上，server3选举为`leader`，server1和server2更新状态为`FOLLOWING`，server3更新状态为`LEADING`

   4. 阶段4：server4启动

      server4启动，发起一次选举。此时server1和server2已经不是`LOOKING状态`，不会更改选票信息。交换选票信息结果为：server3票数为3票，server4票数为4票，此时server4少数服从多数，更改选票信息为server3，并将自己的状态更改为`FOLLOWING`

   5. 阶段5：server5启动

      server5启动，同server4一样当小弟

2. 非第一次启动时的选举机制

   1. 当zookeeper集群中的一台服务器出现以下任意两种情况时，就会进行leader选举

      1. 服务器初始化启动
      2. 服务器运行期间无法和leader通信

   2. 当一台机器进入leader选举流程时，集群可能处于以下两种情况

      1. 集群中的leader节点没有挂掉
      2. 集群中的leader节点挂掉了

      <img src="https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/image.6launj51syw0.png" alt="image" style="zoom:67%;" />

##### ZK集群启动和停止脚本

```java
#!/bin/bash

case $1 in
"start") {
	for i in 172.20.18.163 172.20.18.164 172.20.18.165
	do
		echo -------------- zookeeper $i start -----------------
		ssh $i "/usr/local/zookeeper-3.5.7/bin/zkServer.sh start"
	done
}
;;
"stop") {
	for i in 172.20.18.163 172.20.18.164 172.20.18.165
	do
		echo -------------- zookeeper $i stop -----------------
		ssh $i "/usr/local/zookeeper-3.5.7/bin/zkServer.sh stop"
	done
}
;;
"status") {
	for i in 172.20.18.163 172.20.18.164 172.20.18.165
	do
		echo -------------- zookeeper $i status -----------------
		ssh $i "/usr/local/zookeeper-3.5.7/bin/zkServer.sh status"
	done
}
;;
esac
```

#### 3.2 客户端命令行操作

##### 启动客户端

```shell
# 客户端指定服务端地址启动
root@hadoop103:/usr/local/zookeeper-3.5.7/bin# ./zkCli.sh -server hadoop103
```

##### 命令行语法

| 基本命令  | 功能描述                                                     |
| --------- | ------------------------------------------------------------ |
| ls path   | 查看当前znode 的子节点【可监听】<br />-w 监听子节点变化<br />-s 附加次级信息 |
| create    | 普通创建<br />-s 含有序列<br />-e 临时创建（重启或者超时消失） |
| get path  | 获得节点的值【可监听】<br />-w 监听节点内容变化<br />-s 附加次级信息 |
| set       | 设置节点的具体值                                             |
| stat      | 查看节点状态                                                 |
| delete    | 删除节点                                                     |
| deleteall | 递归删除所有节点                                             |

##### znode节点数据信息

1. 查看当前znode所包含的内容

   ```shell
   [zk: hadoop103(CONNECTED) 3] ls /
   [zookeeper]
   ```

2. 查看当前节点的详细数据

   ```shell
   [zk: hadoop103(CONNECTED) 4] ls -s /
   [zookeeper]cZxid = 0x0
   ctime = Thu Jan 01 00:00:00 UTC 1970
   mZxid = 0x0
   mtime = Thu Jan 01 00:00:00 UTC 1970
   pZxid = 0x0
   cversion = -1
   dataVersion = 0
   aclVersion = 0
   ephemeralOwner = 0x0
   dataLength = 0
   numChildren = 1
   ```

   1. cZxid：创建节点的事物id
   2. ctime：创建节点时的时间戳
   3. mZxid：znode最后更新的事务id
   4. mtime：znode最后更新数据的时间戳
   5. pZxid：znode最后更新的子节点的id
   6. cversion：znode子节点版本号
   7. dataVersion：znode数据版本号
   8. aclVersion：znode访问控制版本号
   9. ephemeralOwner：znode临时节点节点id
   10. dataLength：znode的数据长度
   11. numChildren：znode的子节点数量

##### 节点类型（持久/短暂/有序号/无序号）

> 注意：在分布式操作系统中，顺序号可以被用为所有的事件进行全局排序，这样客户端可以通过顺序号为事件排序

1. 持久类型节点（persistent）

   客户端与服务端断开连接之后，创建的节点还存在

   1. 有序号型持久节点

      ```shell
      [zk: hadoop103(CONNECTED) 25] create -s /chengxuyuan/yanfa/golang
      Created /chengxuyuan/yanfa/golang0000000000
      [zk: hadoop103(CONNECTED) 26] create -s /chengxuyuan/yanfa/golang
      Created /chengxuyuan/yanfa/golang0000000001
      [zk: hadoop103(CONNECTED) 27] ls /chengxuyuan/yanfa
      [golang0000000000, golang0000000001]
      ```

   2. 无序号型持久节点

      ```shell
      [zk: hadoop103(CONNECTED) 10] create /chengxuyuan "java"
      Created /chengxuyua
      
      [zk: hadoop103(CONNECTED) 16] ls /
      [chengxuyuan, sanguo, zookeeper]
      
      [zk: hadoop103(CONNECTED) 18] get -s /chengxuyuan
      java
      cZxid = 0x100000005
      ctime = Wed Aug 04 12:14:40 UTC 2021
      mZxid = 0x100000005
      mtime = Wed Aug 04 12:14:40 UTC 2021
      pZxid = 0x100000005
      cversion = 0
      dataVersion = 0
      aclVersion = 0
      ephemeralOwner = 0x0
      dataLength = 4
      numChildren = 0
      ```

2. 短暂类型节点（Ephemeral）

   客户端与服务端断开连接之后，创建的节点自己删除

   1. 有序号型短暂节点

      ```shell
      [zk: hadoop103(CONNECTED) 5] create -e -s /ephemeral
      Created /ephemeral0000000002
      ```

   2. 无序号型短暂节点

      ```shell
      [zk: hadoop103(CONNECTED) 5] create -e  /ephemeral
      Created /ephemeral
      ```

   3. 修改节点的值

      ```shell
      [zk: hadoop103(CONNECTED) 4] set /chengxuyuan "hangye"
      [zk: hadoop103(CONNECTED) 5] get -s /chengxuyuan
      hangye
      cZxid = 0x100000005
      ctime = Wed Aug 04 12:14:40 UTC 2021
      mZxid = 0x100000011
      mtime = Wed Aug 04 12:32:03 UTC 2021
      pZxid = 0x100000006
      cversion = 1
      dataVersion = 1
      aclVersion = 0
      ephemeralOwner = 0x0
      dataLength = 6
      numChildren = 1
      ```

##### 监听器原理

1. 监听原理详解

   1. 首先创建一个main线程
   2. 在main线程中创建一个zookeeper客户端。其中客户端会创建两个线程，一个是负责网络连接通信的connect线程，一个是负责监听的listener线程
   3. 通过connect线程将注册的监听事件发送给zookeeper
   4. 在zookeeper的监听器列表中将注册的监听器事件添加至列表
   5. zookeeper监听到有数据或者路径变化时，就会将这个消息发送给listener线程
   6. listener线程内部会调用process方法，会对事件做进一步处理

   ![image](https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/image.3ctyim5te3c0.png)

2. 常见的监听事件

   1. 监听节点数据的变化

      ```shell
      get path [watch]
      ```

      1. 在hadoop105上注册监听/chengxuyan节点数据的变化

         ```shell
         [zk: hadoop105:2181(CONNECTED) 3] get -w /chengxuyuan
         hangye
         ```

      2. 在hadoop104上修改/chengxuyan节点的数据

         ```shell
         [zk: hadoop104:2181(CONNECTED) 2] set /chengxuyuan "hangye1"
         ```

      3. 观察 hadoop105 主机收到数据变化的监听

         ```shell
         [zk: localhost:2181(CONNECTED) 4]
         WATCHER::
         
         WatchedEvent state:SyncConnected type:NodeDataChanged path:/chengxuyuan
         ```

         **注意：zookeeper中事件注册一次，只能监听一次。想要再次监听就得在注册一次**

   2. 监听节点增减变化（创建/删除）

      ```shell
      ls path [watch]
      ```

      ```shell
      # 监听/chengxuyuan节点增加变化
      [zk: localhost:2181(CONNECTED) 4] ls  -w /chengxuyuan
      [yanfa]
      
      # 在/chengxuyuan节点创建子节点
      [zk: localhost:2181(CONNECTED) 14] create -s /chengxuyuan/yanfa222
      Created /chengxuyuan/yanfa2220000000001
      
      # 观察到/chengxuyuan节点监听到的变化
      [zk: localhost:2181(CONNECTED) 5]
      WATCHER::
      
      WatchedEvent state:SyncConnected type:NodeChildrenChanged path:/chengxuyuan
      ```

##### 节点删除与查看

1. 删除子节点

   ```shell
   delete /chengxuyuan/python
   ```

2. 递归删除子节点

   ```shell
   deleteall /chengxuyuan
   ```

3. 查看节点状态信息（不查看数据信息）

   ```shell
   stat /chengxuyuan/java
   ```

#### 3.3 客户端API操作

> 前提：保证 hadoop103、hadoop104、hadoop105 服务器上 Zookeeper 集群服务端启动

##### IDEA环境搭建

1. pom文件

   ```xml
   <dependencies>
       <dependency>
           <groupId>junit</groupId>
           <artifactId>junit</artifactId>
           <version>RELEASE</version>
       </dependency>
   
       <dependency>
           <groupId>org.apache.logging.log4j</groupId>
           <artifactId>log4j-core</artifactId>
           <version>2.8.2</version>
       </dependency>
   
       <dependency>
           <groupId>org.apache.zookeeper</groupId>
           <artifactId>zookeeper</artifactId>
           <version>3.5.7</version>
       </dependency>
   </dependencies>
   ```

2. log4j.properties配置文件

   ```properties
   log4j.rootLogger=INFO, stdout
   log4j.appender.stdout=org.apache.log4j.ConsoleAppender
   log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
   log4j.appender.stdout.layout.ConversionPattern=%d %p [%c] - %m%n
   log4j.appender.logfile=org.apache.log4j.FileAppender
   log4j.appender.logfile.File=target/spring.log
   log4j.appender.logfile.layout=org.apache.log4j.PatternLayout
   log4j.appender.logfile.layout.ConversionPattern=%d %p [%c] - %m%n
   ```

3. ZookeeperClient

   ```java
   package com.geek;
   
   import org.apache.zookeeper.*;
   import org.apache.zookeeper.data.ACL;
   import org.junit.Before;
   import org.junit.Test;
   
   import java.io.IOException;
   
   /**
    * @author: lucas.zhao@kuhantech.com
    * @date: 2021/8/4 22:15
    * @description:
    */
   public class ZookeeperClientTest {
   
       private final String connectString = "172.20.18.163:2181,172.20.18.164:2181,172.20.18.165:2181";
   
       private final int sessionTimeout = 2000;
   
       private ZooKeeper zookeeperClient;
   
       @Before
       public void init() throws IOException {
           zookeeperClient = new ZooKeeper(connectString, sessionTimeout, new Watcher() {
               @Override
               public void process(WatchedEvent watchedEvent) {
   
               }
           });
       }
   
       @Test
       public void createNode () throws KeeperException, InterruptedException {
           String createNodeName = zookeeperClient.create("/geek", "lucas.zhao".getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT_SEQUENTIAL);
           System.out.println(createNodeName);
       }
   }
   ```

##### 创建zookeeper客户端

```java
@Before
public void init() throws IOException {
    zookeeperClient = new ZooKeeper(connectString, sessionTimeout, new Watcher() {
        @Override
        public void process(WatchedEvent watchedEvent) {

        }
    });
}
```

##### 创建子节点

```java
@Test
public void createNode () throws KeeperException, InterruptedException {
    String createNodeName = zookeeperClient.create("/geek", "lucas.zhao".getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT_SEQUENTIAL);
    System.out.println(createNodeName);
}
```

##### 获取子节点并监听节点变化

```java
    /**
     * 获取子节点并且监听子节点变化
     * @throws KeeperException
     * @throws InterruptedException
     */
    @Test
    public void getChildrenAndWatch () throws KeeperException, InterruptedException {
        List<String> children = zookeeperClient.getChildren("/", true);
        // children.forEach(System.out::println);
        TimeUnit.SECONDS.sleep(Integer.MAX_VALUE);
    }
```

##### 判断znode是否存在

```java
/**
 * 判断节点是否存在
 */
@Test
public void judgeNodeExist() throws KeeperException, InterruptedException {
    Stat exists = zookeeperClient.exists("/java", false);
    System.out.println(exists == null ? "not exist" : "exist");
}
```

#### 3.4 客户端向服务端写数据流程

1. 写数据之写入请求直接发送给leader节点

   <img src="https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/image.6n5t1witexc0.png" alt="image" style="zoom:50%;" />

2. 写数据之写入请求直接发送给follower节点

   <img src="C:\Users\lucas.zhao\AppData\Roaming\Typora\typora-user-images\image-20210807105435613.png" alt="image-20210807105435613" style="zoom:50%;" />

### 4 服务器动态上下线监听案例

#### 4.1 需求

在分布式系统中，主节点可以有很多台，可以动态上下线，任何一台客户端都可以实时监听到主节点的上下线的变化

#### 4.2 需求分析

<img src="https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/image.1cwqz5yfdpfk.png" alt="image" style="zoom:50%;" />

1. 服务端启动时向zookeeper集群注册信息（这里创建的都是临时节点）
2. 客户端获取到当前在线的服务器列表，并且进行注册监听
3. 客户端1节点下线
4. zookeeper集群向客户端发送服务端下线的通知
5. 客户端的process方法中业务逻辑：重新获取服务器列表，并进行注册监听

#### 4.3 具体实现

DistributeServer

```java
public class DistributeServer {

    private final String connectString = "172.20.18.163:2181,172.20.18.164:2181,172.20.18.165:2181";

    private final int sessionTimeout = 20000;

    private ZooKeeper zooKeeperClient;

    public static void main(String[] args) throws IOException, KeeperException, InterruptedException {
        DistributeServer distributeServer = new DistributeServer();
        // 获取ZK集群
        distributeServer.getConnect();
        // 注册服务器信息到ZK集群
        distributeServer.register(args[0]);
        // 开始执行服务器业务逻辑(睡觉)
        distributeServer.business();
    }

    /**
     * 业务逻辑
     */
    private void business() throws InterruptedException {
        TimeUnit.SECONDS.sleep(Integer.MAX_VALUE);
    }

    /**
     * 注册服务器信息到ZK集群
     */
    private void register(String hostname) throws KeeperException, InterruptedException {
        // 创建临时有序号性节点
        zooKeeperClient.create("/servers/" + hostname, hostname.getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL_SEQUENTIAL);
        System.out.println(hostname + "is online");
    }

    /**
     * 获取ZK集群
     */
    private void getConnect() throws IOException {
        zooKeeperClient = new ZooKeeper(connectString, sessionTimeout, new Watcher() {
            @Override
            public void process(WatchedEvent watchedEvent) {

            }
        });
    }
}
```

DistributeClient

```java
public class DistributeClient {

    private final String connectString = "172.20.18.163:2181,172.20.18.164:2181,172.20.18.165:2181";

    private final int sessionTimeout = 4000;

    private ZooKeeper zooKeeperClient;

    private final String rootNodeName = "/servers";

    public static void main(String[] args) throws IOException, KeeperException, InterruptedException {
        DistributeClient client = new DistributeClient();
        // 获取ZK集群
        client.getConnect();
        // 获取ZK集群中的服务器列表信息
        client.getServerList();
        // 开始执行服务器业务逻辑(睡觉)
        client.business();
    }

    private void getServerList() throws KeeperException, InterruptedException {
        // /servers下面的子节点名称
        List<String> children = zooKeeperClient.getChildren(rootNodeName, true);
        List<String> serverDateList = new ArrayList<>();
        for (String child : children) {
            byte[] data = zooKeeperClient.getData(rootNodeName + "/" + child, false, null);
            serverDateList.add(new String(data));
        }
        System.out.println(serverDateList);
    }

    /**
     * 业务逻辑
     */
    private void business() throws InterruptedException {
        TimeUnit.SECONDS.sleep(Integer.MAX_VALUE);
    }


    /**
     * 获取ZK集群
     */
    private void getConnect() throws IOException {
        zooKeeperClient = new ZooKeeper(connectString, sessionTimeout, new Watcher() {
            @Override
            public void process(WatchedEvent watchedEvent) {
                try {
                    getServerList();
                } catch (KeeperException | InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });
    }
}
```

#### 4.4 测试

### 5 zookeeper分布锁案例

分布式锁区别的Java中普通的锁，分布式锁锁的对象是进程，普通锁锁的对象是线程。

在分布式系统中，比如进程1在访问共享资源，会首先获取锁，进程1获取锁之后会对该资源保持独占，这样可以保证其他的进程无法访问该资源。进程1用完该资源以后将分布式释放掉，让其他的进程进程竞争抢占。通过这个分布式锁机制我们可以保证在分布式系统中多个进程可以有序访问临界资源。因此我们把这个在分布式环境下的锁叫做分布式锁  

#### 5.1 原生zookeeper实现分布锁案例

<img src="https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/image.5hzx9704c0o0.png" alt="image" style="zoom: 33%;" />

1. zookeeper集群在接收到客户端请求之后在 /locks节点下面创建一个临时顺序节点（zookeeper集群默认会选择序列号最小的节点持有分布式锁）
2. 后面的节点会判断自己是不是最小序列号的节点，如果是则获取到锁，如果不是则对前一个节点进行监听
3. 获取到锁，处理完自己的业务之后会delete节点释放锁，然后下面的节点将会收到通知，获取分布式锁

#### 5.2 Curator框架实现分布式锁案例

### ❗6 企业面试真题（面试重点）

#### 6.1 选举机制

#### 6.2 生产集群安装多少zk合适？

#### 6.3 常用命令







