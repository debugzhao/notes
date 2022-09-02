## 第一章 Kafka概述

### 1.1定义

Kafka定义：Kafka是一个开源的分布式事件流平台，用于高性能数据管道、流分析、数据集成和关键任务应用。

发布/订阅：消息的发布者不会直接将消息发送给特定的订阅者，而是将发布的消息分为不同的类别，订阅者只接受感兴趣的消息。

### 1.2 消息队列

Kafka、ActiveMQ、RabbitMQ、RocketMQ

#### 1.2.1传统消息队列的应用场景

1. 缓冲/消峰

   有助于控制和优化数据流经系统的速度，解决生产者消息和消费者处理数据速度不一致的问题。

   <img src="https://img1.imgtp.com/2022/09/02/4TI66nAI.png" alt="消峰.png" style="zoom: 25%;" />

2. 解耦

   允许开发者独立地扩展或者修改两边（生产者、消费者）的处理过程，只要确保他们遵循同样的接口。

   <img src="https://img1.imgtp.com/2022/09/02/ObCWjIJF.png" alt="解耦.png" style="zoom: 25%;" />

3. 异步通信

   允许开发者把一个消费/请求放入队列，但是不会立即处理它，然后在需要的时候再去处理它们。

   <img src="https://img1.imgtp.com/2022/09/02/JHvGR5ib.png" alt="异步通信.png" style="zoom: 25%;" />

#### 1.2.2消息队列的两种通信模式 

1. 点对点通信

   消费者主动拉取数据，收到数据后清除数据

   ![Snipaste_2022-09-02_07-37-47.png](https://img1.imgtp.com/2022/09/02/7SRjClU2.png)

2. 发布订阅方式通信

   可以同时有多个Topic主题、消费者消费数据之后不会删除数据、每个消费者线程互相独立，都可以消费到数据。

   ![Snipaste_2022-09-02_07-38-30.png](https://img1.imgtp.com/2022/09/02/XQK1WcHm.png)

### 1.3 kafka基础架构

![kafka架构.png](https://img1.imgtp.com/2022/09/02/j4YOjJuc.png)

**架构介绍**

1. 为了方便扩展，提高吞吐量一个topic可以分为多个partition
2. 为了配合分区的设计，提出消费组的概念，组内每个消费者group并行消费
3. 为了提高可用性，为每个partition增加若干个副本
4. zookeeper中记录谁是leader，kafka2.8版本以后可以配置不采用zookeeper

**概念介绍**

1. producer

   消息生产者

2. consumer

   消息消费者

3. consumer  group

   消费者组，由多个consumer组成。<font color="red">消费者组内的每个消费者负责消费不同分区的数据，一个分区只能由一个组内的一个消费者消费。所有的消费者组之间互不影响。</font>

   任意一个消费者属于某个消费者组，即消费者组是一个逻辑上的订阅者。

4. broker

   一台kafka服务器就是一个broker。一个集群由多个broker组成。一个broker可以容纳多个topic

5. topic

   可以理解为一个队列，生产者消费者面向的都是一个队列

6. partition

   为了实现扩展性，一个非常大的topic可以拆分成多个partition然后分布到多台broker上。每个partition都是一个有序的队列

7. replica

   为了实现高可用，一个topic的每个partition（分区）都有若干个副本，多个副本构成副本组，每个副本组都有一个Leader和若干个Follower

8. leader

   每个分组的<font color="red">副本组的主</font>，生产者生产数据的对象，消费者消费数据的对象都是leader

9. follower

   每个分区<font color="red">副本组的从</font>，follower会从leader实时同步数据，保持和leader数据的同步。当leader发生故障时，某个follower会成为新的leader





## 第二章 Kafka快速入门

### 2.1安装部署

#### 2.1.1集群规划

#### 2.1.2集群部署

#### 2.1.3集群启动停止脚本

### 2.2Kafka命令行操作

#### 2.2.1主题命令行操作

#### 2.2.2生产者命令行操作

#### 2.2.3消费者命令行操作









#### <font color="red"></font>