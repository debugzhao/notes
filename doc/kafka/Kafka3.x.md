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

    0：生产者发送过来的数据，不需要等待数据落盘，kafka集群就任务数据发送成功

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































#### <font color="red"></font>