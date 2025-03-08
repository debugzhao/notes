# 1 基本概念

- 消费者与消费组

  在Kafka中，多个消费者会形成一个消费组来对主题Topic进行消费，每个消费者会消费不同分区的数据

# 2 消费者基本参数配置

```java
//1.创建配置文件,定义配置信息，其他没有添加的配置为默认配置数据
Properties properties = new Properties();
//key反序列化配置55
properties.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG,StringDeserializer.class.getName());
//value反序列化配置
properties.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG,StringDeserializer.class.getName());
//集群地址
properties.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG,"192.168.10.131:9092");
//消费组信息
properties.put(ConsumerConfig.GROUP_ID_CONFIG,"group.demo");
```

# 3 订阅主题和分区

- 传入一个列表，订阅多个主题

  ```java
  KafkaConsumer<String, String> consumer = new KafkaConsumer<>(properties); consumer.subscribe(Collections.singletonList("test-topic"));
  ```

- 传入正则表达式

  ```java
  //传入一个正则表达式，根据规则订阅主题消息，一旦有新的主题加进来时，消费组会立即进行消费
  consumer.subscribe(Pattern.compile("log*"),new NoOpConsumerRebalanceListener());
  ```

- 订阅指定的主题和分区

  ```java
  //订阅指定的分区、主题
  consumer.assign(Collections.singletonList(new TopicPartition("test-topic",0)));
  ```

# 4 offset消息偏移量

对于Kafka分区而言，每一个消息都有唯一的offset偏移量，用来表示消息在分区中的位置。调用poll()方法时，集群会返回我们没有消费过的消息。当消息从broker返回给消费者时，broker并不会跟踪消息是否被消费者接收到（*可以理解为不可靠的消息传输服务*）

Kafka让消费者自身来管理消息的偏移量，即提供了更新消息偏移量的方法，commit()方法

由于每个消息都有自己的偏移量，可能会带来两个问题：重复消费问题、消息丢失问题

## 4.1 重复消费出现场景<img src="https://i.loli.net/2020/07/26/lBgvsiPrz5IDtqT.png" alt="重复消费" style="zoom:67%;" />

消费者A消费后消息偏移量为2，将偏移量提交后 又来了消费者B开始消费。消费者B消费后消息偏移量为10，如果在消费者A在B提交偏移量之前又进行消费，这时可能出现重复消费的现象。

## 4.2 消息丢失出现场景

当消费者拉取到了分区的某个消息之后，消费者会自动提交了 offset。自动提交的话会有一个问题，试想一下，当消费者刚拿到这个消息准备进行真正消费的时候，突然挂掉了，消息实际上并没有被消费，但是 offset 却被自动提交了。此时这段消息会被认定已经消费过了，也就出现了消息丢失的问题

# 5 消息提交的方式

## 5.1 自动提交

这种方式让消费者来管理消息的偏移量，应用本身不需要进行显式操作。如果没有设置提交的方式时，Kafka默认将提交方式为自动提交，该参数由enable.auto.commit 指定。消费者在调用poll方法后每个5秒钟Kafka提交一次移位，该参数由auto.commit.interval.ms指定。

> 自动提交原理：调用poll方法后 消费者判断是否达到提交时间，如果是则提交上一次poll返回的最大位移。

*注意：如果提交方式设置为自动提交，可能会出现重复消费的问题*

## 5.2 手动同步提交



## 5.2 手动异步提交

## 5.3 再均衡监听器

再均衡是指分区所属的消费者从一个消费者转移到另一个消费者的行为，该机制提高了消费组的高可用性和伸缩性，是我们安全且方便地向消费组内添加或者删除消费者。

*注意：消费者在Kafka再均衡期间是无法拉取消息的*