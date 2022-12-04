## Kafka

1、如何获取 topic 主题的列表

```shell
bash ./bin/kafka-topics.sh --list --bootstrap-server localhost:9092
```

2、生产者和消费者的命令行是什么？

生产者发送消息命令
 ```shell
 bash ./bin/kafka-console-producer.sh --bootstrap-server localhost:9092 --topic first
 > hello
 > test
 ```

消费者消费数据命令

```shell
# --from-beginning 消费所有的历史数据
bash ./bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic first --from-beginning
```

3、consumer 是推还是拉？

consumer采用的是pull模式（消费者主动拉取数据）

pull模式的缺点：如果kafka没有数据，消费者可能会陷入循环中，一直返回空数据。为了解决这种现象，kafka提供了一个参数可以让consumer阻塞直到有新消息到达（或者阻塞consumer直到消息的数量到达指定的数量）

> push模式：
>
> 由broker主动推送数据，kafka没有采用这种方式，是因为由broker决定消息发送速率的话很难适用所有消费者的消费速率

4、讲讲 kafka 维护消费状态跟踪的方法

在kafka中，一个topic可以被分成多个分区，一个partition在同一时刻只能被一个consumer消费，这就意味着partition中消息被消费的位置可以通过一个简单的的整数（offset）跟踪维护。这样就很容易标记每个分区的消费状态

5、[**讲一下主从同步**](https://blog.csdn.net/honglei915/article/details/37565289)

6、为什么需要消息系统，mysql 不能满足需求吗？

1. 缓冲/削峰

   有助于优化数据流经系统的速度，解决生产者消息和消费者处理速度不一致的问题

2. 解耦

   允许开发者独立地扩展、修改两边（生产者/消费者）的数据处理过程，只要是他们遵循同样的接口

3. 异步通信

   很多时候用户不需要立刻处理数据，消息队列提供了异步处理机制，允许开发者把一个数据/请求放入队列中，但是不会立刻处理他们，在需要的时候才会处理他们

7、Zookeeper 对于 Kafka 的作用是什么？

zookeeper主要用在集群中不同节点之间的通信。在kafka中，被用于提交偏移量、leader检测、分布式同步、配置管理、节点状态检测

8、数据传输的事务定义有哪三种？

1. 最多一次

   消息不会被重复发送，最多被传送一次，但是可能漏传

2. 最少一次

   消息不会被漏传，可以传送多次，但是会存在重复发送的情况

3. 精确一次

   消息只会被精确传送一次，不会漏传也不会重复传送

9、Kafka 判断一个节点是否还活着有那两个条件？

1. 节点必须和zookeeper保持通信（zookeeper通过心跳机制判断server是否存活）
2. 如果节点是follower节点，必须及时和leader节点同步写操作，不能延迟太久

10、Kafka和其他MQ对比

1. kafka和RabbitMQ几个角度简单对比

   1. 架构模型方面

      RabbitMQ的broke由exchange、banding、queue组成，其中exchange和banding确定消息的路由键。consumer从queue中获取消息并进行消费。rabbitMQ以broker为中心，有消息确认机制

      kafka遵循一般的MQ结构，由producer、broker、consumer组成。以consumer为中心，消息的消费保存在consumer上，consumer根据消费的点从broker上批量pull数据，没有消息确认机制

   2. 吞吐量方面

      kafka吞吐量很高，内部采用数据批处理机制、零拷贝机制、数据存储在本地磁盘上，批量顺序读写数据。综合以上因素数据处理速度较高

      rabbitMQ吞吐量逊色于kafka，他们的出发点不一样，rabbitMQ支持消息的可靠性传输，支持事务，不支持批量操作

   3. 可用性方面

      rabbitMQ支持mirrot的queue，主queue失效，mirror queue接管

      kafka broker支持主备模式，leader挂掉，从follower中选举出新的leader继续提供服务

   4. 集群负载均衡方面

      kafka采用zookeeper对集群中的broker、producer进行协调管理

      rabbitMQ的负载均衡需要单独的loadbalancer进行支持

> Reference
>
> 1. [kafka和RabbitMQ消息队列对比](https://blog.csdn.net/u013939918/article/details/70947669)

11、讲一讲 kafka的ack的三种机制（broker应答sender线程的三种方式）

0：生产者发送过来数据之后，不需要等待数据落盘，集群就认为数据发送成功

1：生产者发送过来数据之后，leader接收到消息后才认为消息发送成功

-1：生产者发送过来数据之后，leader和ISR队列中的所有节点收到消息后才认为消息发送成功

12、消费者如何不自动提交偏移量，由应用提交

为了我们专注于自己的业务逻辑，kafka提供了自动提交offse功能，但是通过`enable.auto.commit `配置项可以改成`false`，由开发者手动提交offset，其中手动提交又分为`commitSync`（同步提交）和`commitAsync`（异步提交）

```java
ConsumerRecords<> records = consumer.poll();
for (ConsumerRecord<> record : records){
    // ......
    tyr{
        consumer.commitSync()
    }
    // ......
}
```

13、消费者故障，出现活锁问题如何解决？   

14、如何控制消费的位置

1. 指定offset开始消费

   kafka 使用`seek(TopicPartition, long)`指定新的消费位置，用于查找服务器保留的最早和最新的 offset 的特殊的方法也可用`seekToBeginning(Collection)` 和`seekToEnd(Collection)）`

   ```java
   // 遍历所有分区，并指定offset从1700的位置开始消费
   for (TopicPartition tp: assignment) {
     kafkaConsumer.seek(tp, 1700);
   }
   ```

2. 指定时间开始消费

15、**kafka 分布式（不是单机）的情况下，如何保证消息的顺序消费?**

针对消息的顺序性需要从 producer 和 consumer两个角度来考虑；面对有顺序消费业务的需求，还分为全局有序和局部有序

1. 全局有序

   一个 topic下的所有消息都需要按照生产顺序去消费

2. 局部有序

   一个 topic 下的消息，只需要满足针对指定业务字段的生产顺序消费。例如 topic的消息是订单流水表，只需要针对订单 id 的生产顺序消费即可

**全局有序**

一个 topic 可以由多个 partition 组成，当生产者按照顺序发送消息到 kafka 集群时，消息可能会发送到不同的 partition，此时消费者再去消费就会出现乱序现象。因此要想 topic 全局有序，一个 topic 只能对应一个 partition。

并且对应的消费者应该使用单线程或者可以保证消息顺序线程模型来消费，否则消费端还是会出现消费乱序。

**局部有序**

要满足局部有序，只需要在发消息的时候指定 partition key，kafka 对其进行 hash运算，然后根据运算结果决定将消息放到对应的 partition中。这样 partition key 相同的消息就会被放入同一个 partition 中。

此时topic的 partiton数量仍然可以设置为多个，可以提升 topic 总体的吞吐量。

**消息重试对顺序消费的影响**

对于一个有着先后顺序的消息 AB，正常情况下是先发送 A 再发送 B；但是在异常情况下A 发送失败了，B 发送成功了，由于重试机制A 在 B 发送成功以后再次重试发送成功。此时原本正常情况的 AB 变成了 BA

这种情况下，严格的顺序消费还需要将`max.in.flight.requests.per.connection`参数配置为 1。这个参数控制着生产者在服务器响应之前可以发送多少条消息，它的值越高内存占用就越高，同时也会提高系统的吞吐量。把它设置为 1 就可以使消息按照发送的顺序写入服务器

 把`max.in.flight.requests.per.connection` 配置为 1 会严重降低系统吞吐量，在某些业务场景下是不能接受的。如果放弃使用失败重试机制，可以在消费端增加失败标记的记录，然后用定时任务轮询重试失败的消息，并做好监控报警

16、**kafka 的高可用机制是什么？**



17、**kafka 如何减少数据丢失**

18、**kafka 如何不消费重复数据？**比如扣款，我们不能重复的扣。

19、kafka partitions分区路由规则



引用

- [18道kafka题哪些你还不会](https://developer.aliyun.com/article/740170)
- [一文理解Kafka如何保证消息顺序性](https://cloud.tencent.com/developer/article/1839597)



1. 零拷贝