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

## ~~第二章 Kafka快速入门~~

### ~~2.1安装部署~~

#### ~~2.1.1集群规划~~

#### ~~2.1.2集群部署~~

#### ~~2.1.3集群启动停止脚本~~

### ~~2.2Kafka命令行操作~~

#### ~~2.2.1主题命令行操作~~

#### ~~2.2.2生产者命令行操作~~

#### ~~2.2.3消费者命令行操作~~

## 第三章Kafka生产者

### 3.1生产者消息发送流程

在消费的发送过程中涉及到两个线程，分别是main线程和sender线程。在main线层中会创建一个双端队列Record Accumulator，main线程将消息发送给RecordAccumulatot，sender线程会不断从队列中拉去消息发送到kafka broker。

![kafka生产者消息发送流程.png](https://img1.imgtp.com/2022/09/06/cmKkhL7x.png)

1. 分区器

   分区器负责将消息发往不同分区

2. record accumulator

   双端队列的默认大小是32MB，其中每个分区细分batch，batch size达到16KB，sender线程才会从双端队列中拉取消息。

   1. batch.size

      只有数据积累到batch.size之后，sender线程才会发送消息，默认是16KB

   2. Linger.ms

      如果数据迟迟没有累积到batch.size，sender线程会等待linger.ms设置的时间，时间到了之后就会发送消息。单位是ms，默认值是0ms，表示没有延迟。

3.  Ack应答机制 

    0：生产者发送过来的数据，不需要等待数据落盘，kafka集群就认为数据发送成功

    1：生产者发送过来的数据，leader收到数据后应答才算数据发送成功。

    -1(all)：生产者发送过来的数据，leader和ISR队列（follower）里的所有节点收到数据之后才算数据发送成功

### 3.2异步发送API

```java
public class ProducerCallback {
    public static void main(String[] args) {
        // 配置
        Properties properties = new Properties();
        properties.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "10.211.55.8:9092,10.211.55.9:9092,10.211.55.10:9092");
        properties.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
        properties.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
        // 创建kafka生产者对象
        KafkaProducer<String, String> kafkaProducer = new KafkaProducer<>(properties);
        // 发送消息
        try {
            kafkaProducer.send(new ProducerRecord<>("first", "hello kafka!!!!!!!!"), new Callback() {
                @Override
                public void onCompletion(RecordMetadata recordMetadata, Exception e) {
                    if (e == null) {
                        System.out.println("主题：" + recordMetadata.topic() + " 分区：" + recordMetadata.partition());
                    } else {
                        e.printStackTrace();
                    }
                }
            });
        } finally {
            kafkaProducer.close();
        }
    }
}
```

### 3.3同步发送API

### 3.4生产者分区

#### 分区的好处

1. 便于合理使用分区资源

    通过合理使用分区的，可以实现负载均衡的效果

2. 提高并行度

   生产者可以以分区发送数据，消费者可以以分区消费数据，提高了数据发送、消费的并行度。

#### 分区策略

1. 在指明partition的情况下，数据直接写入指定分区
2. 在没有指定partition但是有key的情况下，数据写入hash(key) %  partitionNum的分区中
3. 没有指定partition并且没有key的情况下，kafka会随机选择一个分区写入数据，直到这个分区被写满为止

#### 自定义分区策略

```java
public class MyPartition implements Partitioner {
    @Override
    public int partition(String topic, Object key, byte[] keyBytes, Object value, byte[] valueBytes, Cluster cluster) {
        String message = value.toString();
        int partition;
        if (message.contains("usa")) {
            partition = 0;
        } else {
            partition = 1;
        }
        return partition;
    }

    @Override
    public void close() {

    }

    @Override
    public void configure(Map<String, ?> map) {

    }
}
```

### 3.5生产经验 生产者如何提高吞吐量

1. batch.size：批次大小，默认16k 
2. linger.ms：等待时间，修改为5-100ms
3. compression.type：压缩snappy
4. RecordAccumulator：缓冲区大小，修改为64m

### 3.6生产经验 数据可靠性

1. ack = 0

   可靠性分析：丢数据

   生产者发送过来的数据，不需要等待数据落盘就认为消息发送成功。如果leader节点收到的数据还在内存中，数据还没有落盘或者还没有来得及和follower节点同步，此时leader节点挂掉数据会丢失。

2. ack = 1

   可靠性分析：丢数据

   生产者发送过来的数据，leader节点收到数据后（落盘）认为消息发送成功。如果leader节点应答完成之后还没有和follower节点同步数据挂掉，此时kafka集群会重新选举新的leader节点，新的leader节点不会再次收到之前的数据，因为在上一轮消息发送过程中生产者已经认为消息发送成功了。<font color="red">所以最终新的leade节点和其他的follower节点会丢失数据</font>

   <img src="https://img1.imgtp.com/2022/09/08/p64Ct5hF.png" alt="ack_1.png" style="zoom:33%;" />

3. ack = -1

   生产者发送过来的数据，leader节点和ISR队列中的所有节点收到数据后才消息发送成功

   <img src="https://img1.imgtp.com/2022/09/08/Xqwcci66.png" alt="ack_-1.png" style="zoom:33%;" />

   Leader收到数据，所有Follower都开始同步数据，但有一个Follower，因为某种故障，迟迟不能与Leader进行同步，那这个问题怎么解决呢？

   Leader维护了一个动态的in-sync replica set（ISR），意为和Leader保持同步的Follower+Leader集合(leader：0，isr:0,1,2)。如果Follower长时间未向Leader发送通信请求或同步数据，则该Follower将被踢出ISR。该时间阈值由replica.lag.time.max.ms参数设定，默认30s。例如2超时，(leader:0, isr:0,1)。这样就不用等长期联系不上或者已经故障的节点。

   <font color="red">数据完全可靠条件 = ACK级别设置为-1 + 分区副本大于等于2 + ISR里应答的最小副本数量大于等于2</font>

### 3.7生产经验 数据去重

#### 数据传递的语义

1. 数据至少发送一次

   ack = -1 + 分区副本书 > 2  + ISR队列中follower数量 > 2

2. 数据最多发送一次

    ack = 0

3. 精确一次

   对于一些非常重要的数据（和金钱相关的数据），要求数据即不能丢失也不能重复。<font color="red">因此通过幂等性操作 + 事物操作可以保证数据去重。</font>

 总结：

at least one：可以保证数据不丢失，但是不能保证数据不重复

at most one：可以保证数据不重复，但是不能保证数据不丢失

#### 幂等性

1. 幂等性原理

   幂等性就是无论producer向broker发送多少条重复数据，broker端都是会持久化一条数据。

   <font color="red">精确数据发送一次：幂等性 + 至少一次（ack = -1 +  分区副本数 >= 2 + ISR队列中follower数量 >= 2）</font>

   重复数据的判断标准：具有<PID,partitionID,SeqNumber>相同主键的消息提交是，broker只会提交一条，其中PID在kafka进程每次重启之后都会生成一个新的id，<font color="red">所有幂等性只能保证在单分区单会话内消息不会重复，不能满足我们的需求。</font>

   <img src="https://img1.imgtp.com/2022/09/09/uQgIBnm9.png" alt="幂等性原理.png" style="zoom:67%;" />

2. 开启幂等性

   ```yaml
   enable.idempotence: true
   ```

#### 事物

 开启事物之前必须开启幂等性，只有幂等性+事物操作才可以保证在kafka集群中（多会话场景中）数据的不重复。

producer在使用事物之前必须先指定唯一的事物ID，有了事物id即使客户端挂了，在重启之后也能保证继续恢复之前的事物。

```java
public class Producer04Trancaction {
    public static void main(String[] args) {
        // 配置
        Properties properties = new Properties();
        properties.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "10.211.55.8:9092,10.211.55.9:9092,10.211.55.10:9092");
        properties.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
        properties.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
        properties.put(ProducerConfig.TRANSACTIONAL_ID_CONFIG, "transaction_id_01");

        // 创建kafka生产者对象
        KafkaProducer<String, String> kafkaProducer = new KafkaProducer<>(properties);
        // 初始化事物
        kafkaProducer.initTransactions();
        // 开启事物
        kafkaProducer.beginTransaction();
        // 发送消息
        try {
            kafkaProducer.send(new ProducerRecord<>("first", "test transaction!!!"));
            int a = 1 / 0;
            // 提交事物
            kafkaProducer.commitTransaction();
        } catch (Exception e) {
            // 出现异常终止事物
            kafkaProducer.abortTransaction();
        } finally {
            kafkaProducer.close(Duration.ofMillis(3000));
        }
    }
}
```

### 3.8生产经验 数据有序

<img src="https://img1.imgtp.com/2022/09/09/uql1cwWh.png" alt="数据有序.png" style="zoom: 33%;" />

1. 单分区数据可以有序（通过排序的方式实现数据有序）
2. 多分区、分区与分区之间数据是无序的

### 3.9生产经验 数据乱序

Kafka1.x以后的版本保证数据单分区有序，条件如下：

1. 未开启幂等性

   ```properties
   # 需要设置为1
   max.in.flight.requests.per.connection = 1
   ```

2. 开启幂等性

   ```properties
   # 需要设置小于等于5
   max.in.flight.requests.per.connection <=5
   ```

   开启幂等性之后kafka broker会缓存producer发来的最近5个request的元数据，所以无论如何都可以保证最近五个requst的数据是有序的。

## 第四章 Kafka Broker

### 4.1 Broker工作流程

#### 4.1.1 Zookeeper存储kafka信息

<img src="https://img1.imgtp.com/2022/09/14/2FmE58HC.png" alt=" zk中存储的kafka信心.png" style="zoom:33%;" />

1. 记录有哪些服务器信息
2. 记录谁是leader，有哪些服务器可用
3.  辅助选举leader信息

#### 4.1.2 Kafka总体工作流程

<img src="https://img1.imgtp.com/2022/09/14/APYNKms3.png" alt="broker总体工作流程.png" style="zoom: 33%;" />

1. broker启动之后在zk中注册信息

2. 每个broker中都有controller节点，zk中维护controller节点，在zk节点中controller谁先注册谁说了算

3. 由选举出来的controller监听brokers节点的变化

4. controller决定leader选举

   选举规则：再ISR队列中存活为前提，按照AR中排在前面的优先。例如AR[1,0,2]，ISR为[0,1,2]，那么leader就会按照[1,0,2]的顺序轮询

5. controller将leader节点、follower节点信息上传到zk

6.  其他controller节点从zk中同步信息

7.  假设leader节点挂掉了

8. controller监听到节点发生了变化，获取最新的ISR

9. zk中的controller获取最新的ISR

10. 选举出新的leader（在ISR队列中为前提，按照AR中排在前面的优先）

11. 将最新的leader信息、follower信息更新到zk中

#### 4.1.3 Broker重要参数

### 4.2  生产经验（扩容&缩容）

### 4.3 Kafka副本

#### 4.3.1 副本基本信息

1. kafka副本作用：提高数据可靠性

2. kafka默认副本一个，生产环境副本数量一般配置为两个，保证数据的可靠性。太多的副本反而会增加磁盘的存储空间，降低效率。

3. kafka副本分为leader和follower。kafka生产者之后给leader副本发数据，然后follower副本找leader副本同步数据。

4. kafka分区中所有的副本统称为AR（Assigned Repllicas） AR = ISR + OSR

   ISR：表示和leader保持同步的follower集合，如果follower副本长时间没有向leader副本所在节点通信或者同步数据，leader将该follower从ISR队列中剔除。该阈值由***\*replica.lag.time.max.ms\**** 配置。

   leader发生故障之后，将会从ISR队列中选取新的leader。

   OSR：表示follower与leader同步时，延迟过多的副本

#### 4.3.2 Leader选举流程

#### 4.3.3 Leader和Follower故障处理细节

**LEO（Log  End Offset）：每个副本中最后一个offset + 1**

**HW（High watermark）：所有副本中最小的LEO**

 **消费者能够看到的最小的offset是HW -1，也是分区副本组中最小的offset**

![1663403501473.png](https://img1.imgtp.com/2022/09/17/RNqv9Yee.png)

故障情况：

1. follower 故障
   1. follower故障之后会被剔除ISR队列
   2.  此时其他的follower分区和leader分区还会接受数据
   3.  等故障的follower恢复之后，follower会读取本读磁盘记录的上次的HW，并将log文件中高于HW部分的数据截取掉，从上次的HW处开始从leader同步数据。

2. leader故障

   <img src="https://img1.imgtp.com/2022/09/19/GFrHLwDi.png" alt="leader故障处理细节.png" style="zoom:33%;" />

   1. leader发生故障之后会从ISR队列中选出一个新的leader
   2. 为保证所有副本之间的数据一致性，其余的follower会先将各自log文件中高于HW的数据截掉，然后从新的leader中同步数据

    *这只能保证所有副本之间数据的一致性，不能保证数据不丢失或者不重复*

#### 4.3.4 分区副本分配

如果kafka服务器只有4个节点，那么设置kafka的分区数大于服务器台数，在kafka底层如何分配存储副本呢？

1. 创建16分区，3个副本，名称为second的topic

   ```shell
   [atguigu@hadoop102 kafka]$ bin/kafka-topics.sh --bootstrap-server hadoop102:9092 --create --partitions 16 --replication-factor 3 --topic second
   ```

2. 查看分区和副本情况

   ```shell
   [atguigu@hadoop102 kafka]$ bin/kafka-topics.sh --bootstrap-server hadoop102:9092 --describe --topic second
   
   Topic: second4	Partition: 0	Leader: 0	Replicas: 0,1,2	Isr: 0,1,2
   Topic: second4	Partition: 1	Leader: 1	Replicas: 1,2,3	Isr: 1,2,3
   Topic: second4	Partition: 2	Leader: 2	Replicas: 2,3,0	Isr: 2,3,0
   Topic: second4	Partition: 3	Leader: 3	Replicas: 3,0,1	Isr: 3,0,1
   
   Topic: second4	Partition: 4	Leader: 0	Replicas: 0,2,3	Isr: 0,2,3
   Topic: second4	Partition: 5	Leader: 1	Replicas: 1,3,0	Isr: 1,3,0
   Topic: second4	Partition: 6	Leader: 2	Replicas: 2,0,1	Isr: 2,0,1
   Topic: second4	Partition: 7	Leader: 3	Replicas: 3,1,2	Isr: 3,1,2
   
   Topic: second4	Partition: 8	Leader: 0	Replicas: 0,3,1	Isr: 0,3,1
   Topic: second4	Partition: 9	Leader: 1	Replicas: 1,0,2	Isr: 1,0,2
   Topic: second4	Partition: 10	Leader: 2	Replicas: 2,1,3	Isr: 2,1,3
   Topic: second4	Partition: 11	Leader: 3	Replicas: 3,2,0	Isr: 3,2,0
   
   Topic: second4	Partition: 12	Leader: 0	Replicas: 0,1,2	Isr: 0,1,2
   Topic: second4	Partition: 13	Leader: 1	Replicas: 1,2,3	Isr: 1,2,3
   Topic: second4	Partition: 14	Leader: 2	Replicas: 2,3,0	Isr: 2,3,0
   Topic: second4	Partition: 15	Leader: 3	Replicas: 3,0,1	Isr: 3,0,1
   ```

   <img src="https://img1.imgtp.com/2022/09/19/WWREXXnr.png" alt="分区副本分配.png" style="zoom: 33%;" />

#### 4.3.5 生产经验 手动调整分区副本存储

在生产环境中，每台服务器的配置和性能不一致，但是Kafka只会根据自己的代码规则创建对应的分区副本，就会导致个别服务器存储压力较大。所有需要手动调整分区副本的存储。

需求：创建一个新的topic，4个分区，两个副本，名称为three。将该topic的所有副本都存储到broker0和broker1两台服务器上。

<img src="https://img1.imgtp.com/2022/09/19/MYktYrQb.png" alt=" 调整分区副本存储.png" style="zoom: 33%;" />

1. 创建一个新的topic，名称为three

   ```shell
   [atguigu@hadoop102 kafka]$ bin/kafka-topics.sh --bootstrap-server hadoop102:9092 --create --partitions 4 --replication-factor 2 --topic three
   ```

2. 查看分区副本存储情况

   ```shell
   [atguigu@hadoop102 kafka]$ bin/kafka-topics.sh --bootstrap-server hadoop102:9092 --describe --topic three
   ```

3. 创建新的副本存储计划

   ```shell
   vim increase-replication-factor.json
   
   {
   "version":1,
   "partitions":[{"topic":"three","partition":0,"replicas":[0,1]},
   {"topic":"three","partition":1,"replicas":[0,1]},
   {"topic":"three","partition":2,"replicas":[1,0]},
   {"topic":"three","partition":3,"replicas":[1,0]}]
   }
   ```

4. 执行副本存储计划

   ```shell
   [atguigu@hadoop102 kafka]$ bin/kafka-reassign-partitions.sh --
   bootstrap-server hadoop102:9092 --reassignment-json-file 
   increase-replication-factor.json --execute
   ```

5. 验证副本存储计划

   ```shell
   [atguigu@hadoop102 kafka]$ bin/kafka-reassign-partitions.sh --
   bootstrap-server hadoop102:9092 --reassignment-json-file 
   increase-replication-factor.json --verify
   ```

#### 4.3.6 生产经验 partiton的leader负载均衡

正常情况下，Kafka本身会自动把Leader Partition均匀分散在各个机器上，来保证每台机器的读写吞吐量都是均匀的。但是如果某些broker宕机，会导致Leader Partition过于集中在其他少部分几台broker上，这会导致少数几台broker的读写请求压力过高，其他宕机的broker重启之后都是follower partition，读写请求很低，造成集群负载不均衡。

<img src="https://cdn.staticaly.com/gh/Andre235/-community@master/src/image.6u3bcg1n7vs0.webp" alt="image" style="zoom: 50%;" />

关于partition的leader自动均衡的三个参数

1. auto.leader.rebalance.enable

   默认是true。自动Leader Partition 平衡

2. leader.imbalance.per.broker.percentage

   默认是10%。每个broker允许的不平衡的leader的比率。如果每个broker超过了这个值，控制器会触发leader的平衡

3. leader.imbalance.check.interval.seconds

   默认值300秒。检查leader负载是否平衡的间隔时间。

#### 4.3.7 生产经验 增加副本因子

### 4.4 文件存储

#### 4.4.1文件存储机制

 从文件存储的角度而言，topic是逻辑上的概念，<font color="red">partition是物理上的概念</font> ，每个partition对应一个log文件（实际上这里的log文件也是逻辑概念），该log 文件存储的就是producer生产的数据（<font color="red">producer生产的数据会追加写到log文件末尾</font> ）。为了防止log文件过大导致数据定位效率低下，kafka采用<font color="red">分片</font> 和<font color="red"> 索引</font>的方式定位数据，<font color="red">将每个partition分为多个segment</font> （每个segment默认为1GB大小），每个segment包括：<font color="red"> .index文件、.log文件、.timeindex文件</font>。这些文件位于一个文件夹中，文件夹的命名规则为topic名称+ 分区号，例如first-0。

> 一个topic会分为多个partition
>
> 一个partition会切分成多个segment
>
>  .log文件：日志文件
>
> .index文件： 偏移量索引文件
>
> .timeindex文件： 时间戳索引文件 []()
>
> <font color="red"> .index文件和.log文件是以当前segment文件的第一条消息的offset命名的</font> 

<img src="https://cdn.staticaly.com/gh/Andre235/-community@master/src/image.4g530n25eqy0.webp" alt="image" style="zoom: 33%;" />

**kafka log索引注意事项**

1. index为<font color="red">稀疏索引</font>，大约每往日志文件中写4kb的数据，会往索引文件中写入一条索引

   ```shell
   log.index.interval.bytes默认大小为4KB
   ```

2.  index文件中保存的offset为相对offset，可以将offset值控制在固定大小范围内，这样可以确保offset的值占用的空间不会太大

**kafka log文件索引数据流程**

1. 根据目标offset定位到segment文件
2. 找到小于等于目标offset的最大offset对应的索引项
3. 定位到log文件
4. 根据position向下遍历，找到对应的record

#### 4.4.2文件清除策略

kafka保存日志的时间默认为7天。

 kafka提供了两种日志清除策略，分别是delete删除策略、 compact压缩策略

1. 删除策略（默认策略）

   将过期数据删除，以segment中所有记录中的最大时间戳作为该文件时间戳

2. 压缩策略

   <font color="red">对于相同key的不同value值，只保留最后一个版本。</font> 

   <font color="red">这种策略只适合特殊场景，比如消息的key是用户ID，value是用户的资料，通过这种压缩策略，整个消息集里就保存了所有用户最新的资料。</font> 

### 4.5 高效读写数据

Kafka集群高效读写数据的原因：

1. kafka本身是分布式集群，利用分区技术（一个topic划分成多个partition），提高了系统的并行度、吞吐量

2. 读数据采用的是稀疏索引，可以快速定位到要消费的数据

3. 顺序写磁盘（追加写数据），顺序写效率之所高是因为省去了大量磁头寻址的时间

4. 零拷贝

   Kafka的数据加工处理操作交由Kafka生产者和Kafka消费者处理。Kafka Broker应用层不关心存储的数据，所以就不用走应用层，传输效率高

## 第五章 Kafka消费者

### 5.1 kafka消费方式

<img src="https://cdn.staticaly.com/gh/Andre235/-community@master/src/image.3l8585xmp4e0.webp" alt="image" style="zoom: 33%;" />

1. pull模式

   由消费者主动拉取数据，<font color="red">kafka采用的是pull模式。</font>

   pull模式的缺点：如果kafka没有数据，消费者可能会陷入循环之中，一直返回空数据

2. push模式

    由broker主动推送数据

   Kafka没有采用这种方式，因为由broker决定消息发送速率，很难适应所有消费者的消费速率

### 5.2 Kafka消费者工作流程

#### 5.2.1 消费者工作流程

<img src="https://cdn.staticaly.com/gh/Andre235/-community@master/src/image.75nmcrzfbfk0.webp" alt="image" style="zoom: 50%;" />

<font color="red">消费者与消费者之间是完全独立的， 因此多个消费者可以消费同一个分区数据。</font> 

一个消费者可以消费多个分区的数据

每个分区的数据只能由消费者组的一个消费者消费

每个消费者的offset由消费者提交到系统主题（_consumer_offsets）里面保存

#### 5.2.2 消费者组原理

消费者组（Consumer Group）： 由多个groupid相同的消费者构成一个消费者组 

1. 消费者组内的每个消费者负责消费不同分区的数据
2. 消费者组之间互相独立，每个消费者组是一个逻辑上的订阅者

#### 5.2.3 消费者工作原理

### 5.3 消费者API

#### 5.3.1 消费者消费数据

```java
public class CustomerConsumer01 {
    public static void main(String[] args) {

        // 1.配置
        Properties properties = new Properties();
        properties.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "10.211.55.8:9092,10.211.55.9:9092,10.211.55.10:9092");
        properties.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
        properties.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
        properties.put(ConsumerConfig.GROUP_ID_CONFIG, "test");

        // 2.创建消费者
        KafkaConsumer<String, String> consumer = new KafkaConsumer<>(properties);

        // 订阅主题
        ArrayList<String> topics = new ArrayList<>();
        topics.add("first");
        consumer.subscribe(topics);

        // 3.消费消息
        while (true) {
            ConsumerRecords<String, String> records = consumer.poll(Duration.ofSeconds(1));
            for (ConsumerRecord<String, String> record : records) {
                System.out.println(record);
            }
        }
    }
}
```

#### 5.3.2 指定分区消费数据

```java
// 指定要消费的分区
ArrayList<TopicPartition> topicPartitions = new ArrayList<>();
topicPartitions.add(new TopicPartition("first", 0));
consumer.assign(topicPartitions);
```

### 5.4 分区分配以及再平衡

> 注意： 通过命令行的方式操作分区只能增加不能减少

**分区分配的背景**：一个消费者组中由多个consumer组成，一个topic中有多个partition组成，<font color="red">现在的问题是到底由哪个consumer消费哪个partition？这就是分区分配策略要考虑的事情</font> 

kafka有四种主流的分区分配策略： Range、RoundRobin、Sticky、CooperativeSticky。 可以通过参数partition.assignment.strategy 配置项修改分区分配策略。默认的分区分配策略为：Range + CooperativeSticky（kafka可以同时配置多个分区分配策略）

#### Range分区策略以及再均衡（根据partition范围进行分区）

<font color="red">**Range分区策略针对的是某一个分区**</font> 

1. Range分区策略原理

   <img src="https://cdn.staticaly.com/gh/Andre235/-community@master/src/image.h9irgjx8cds.webp" alt="image" style="zoom: 50%;" />

2. Range分区策略再均衡案例

   停止掉0号消费者，快速重新发送消息观看结果（45s以内，越快越好）

   1号消费者：消费到3、4号分区数据

   2号消费者：消费到5、6号分区数据

   0号消费者的任务会整体被分配到1号消费者或者2号消费者。

   说明：0号消费者挂掉后，消费者组需要按照超时时间45s来判断它是否退出，所以需要等待，时间到了45s后，判断它真的退出就会把任务分配给其他broker执行。

3. Range分区策略缺点 

   <font color="red">容易产生数据倾斜</font> 

#### RoundRobin分区策略以及再均衡（轮询方式进行分区）

RoundRobin分区策略是针对集群中所有的topic而言的。RoundRobin轮询分区策略，是把集群中所有的partition和所有的消费者列出来，然后按照hashcode进行排序，最后通过轮询算法分配patititon给到各个消费者。

<img src="https://cdn.staticaly.com/gh/Andre235/-community@master/src/image.5s9nmrohafc0.webp" alt="image" style="zoom:50%;" />

#### Sticky分区策略以及再均衡

粘性分区分配策略：可以理解为分配的结果带有“粘性的”。即在执行一次新的分配之前，考虑上一次分配的结果，尽量少的调整分配的变动，可以节省大量的开销。

### 5.5 offset位移

#### offset的默认位置

<font color="red">在kafka0.9版本之前consumer默认将offset保存在zookeeper中，在0.9版本之后offset保存在一个__consumer_offset的系统内置topic中</font> 

#### 自动提交offset

为了使我们能够专注于自己的业务逻辑，Kafka提供了自动提交offset的功能。

`enable.auto.commit`：是否开启自动提交offset功能，默认是true

#### 手动提交offset

1. 同步提交offset（commitSync）

   必须等待offset提交完毕之后，再去消费下一批数据（会阻塞当前消费者线程）

2. 异步提交offset（commitAsync）

   发送完提交offset请求之后（不会管请求是否处理完毕），就开始处理下一批数据

#### 指定offset消费

#### 指定时间消费

#### 漏消费和重复消费

1. 重复消费（自动提交offset引起的）
   1. consumer每隔5s自动提交offset
   2. 如果提交offset后的2s，consumer挂掉
   3. 再次重启consumer，会从上一次提交的offset处继续消费，会导致重复消费
2. 漏消费（手动提交offset）
   1. 设置手动提交offset
   2. 当offset被提交后，数据还在consumer内存中还没有落盘
   3. 此时消费者线程被kill掉，那么offset已经被提交但是数据还没有被处理，导致内存中的数据都是（漏消费）

### 5.6 生产经验 消费者事务

如果consumer端完成精确一次性消费，则需要consumer端将消费过程和提交offset做原子绑定。

### 5.7 生产经验 数据积压（如何提高消费者吞吐量）

1. 如果是kafka消费能力不足

   则可以考虑增加topic的分区数，同时提升消费者组的消费者消费者数量（消费者数 = 分区数量）。二者缺一不可

2. 如果是下游的数据处理不及时

   提神每批次拉去的数据量，拉去的数据量过少，使得处理的数据量 <  生产的数据量也会造成数据积压

# Kafka生产调优手册

## 1.生产者调优

### 1.1 生产者核心参数配置

1.2 生产者如何提高吞吐量

1.3 如何保证数据可靠性

1.4 数据如何去重

1.5 数据乱序如何解决

2.broker调优

2.1broker核心参数配置

2.2broker扩容/缩容

2.3增加分区

2.4增加副本因子

2.5手动调整分区副本存储

2.6Leader Partition负载均衡

2.7自动创建主题

3.消费者调优

3.1消费者核心参数配置

3.2消费者再均衡

3.3指定offset消费

3.4指定时间消费

3.5消费者事务

3.6消费者如何提高吞吐量

4.总体调优

4.1如何提高吞吐量

4.2数据精准一次

4.3合理设置分区数

4.4单条日志大于1M

4.5服务器挂掉如何处理

4.6集群压力测试







#### <font color="red"></font> 
