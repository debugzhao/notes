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

##### 选举机制（面试重点）

##### ZK集群启动和停止脚本

#### 3.2 客户端命令行操作

##### 命令行语法

##### znode节点数据类型

##### 节点类型（持久/短暂/有序号/无序号）

##### 监听器原理

##### 节点删除与查看

#### 3.3 客户端API操作

##### IDEA环境搭建

##### 创建zookeeper客户端

##### 创建子节点

##### 获取子节点并监听节点变化

##### 判断znode是否存在

#### 3.4 客户端向服务端写数据流程

### 4 服务器动态上下线监听案例

#### 4.1 需求

#### 4.2 需求分析

#### 4.3 具体实现

#### 4.4 测试

### 5 zookeeper分布锁案例

#### 5.1 原生zookeeper实现分布锁案例

#### 5.2 Curator框架实现分布式锁案例

### 6 企业面试真题（面试重点）

#### 6.1 选举机制

#### 6.2 生产集群安装多少zk合适？

#### 6.3 常用命令







