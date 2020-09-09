#### 1.1 常见概念

- AR(Assigned Replicas)

  分区中的所有副本统称为AR

- ISR(In-Sync-Replicas)

  所有与Leader部分保持同步的副本称为ISR

- OSR(Out-of-Sync-Replicas)

  与Leader副本同步滞后过多的副本称为OSR

- HW(High Watermark)

  高水位，表示了一个特定的offset(偏移量)，消费者只能拉取到这个offset之前的消息

#### 1.2Kafka常用命令

- 启动zookeeper服务

  ```shell
  [root@ES01 zookeeper-3.4.14]# bin/zkServer.sh start
  ZooKeeper JMX enabled by default
  Using config: /opt/zookeeper/zookeeper-3.4.14/bin/../conf/zoo.cfg
  Starting zookeeper ... STARTED
  ```

- 查看是否启动成功

  ```shell
  [root@ES01 zookeeper-3.4.14]# jps -l
  4599 org.apache.zookeeper.server.quorum.QuorumPeerMain
  2103 /usr/lib/jenkins/jenkins.war
  4637 sun.tools.jps.Jps
  ```

- 启动Kafka服务

  ```powershell
  [root@ES01 kafka_2.12-2.3.0]# bin/kafka-server-start.sh config/server.properties 
  ```

- 创建主题：

  ```shell
  bin/kafka-topics.sh --zookeeper localhost:2181 --create --topic log --partitions 2 --replication-factor 1
  ```

  > 参数介绍：
  >
  > --zookeeper 连接zookeeper服务地址
  >
  > --create --topic 执行的操作为创建主题
  >
  > --partitions 主体内分区的个数
  >
  > --replication-factor 备份的数量

- 查看所有创建成功的主题

  ```shell
  bin/kafka-topics.sh --zookeeper localhost:2181 --list
  ```

- 查看某一个主题详情

  ```shell
  [root@ES01 kafka]# bin/kafka-topics.sh --zookeeper localhost:2181 --describe --topic log
  Topic:log	PartitionCount:2	ReplicationFactor:1	Configs:
  	Topic: log	Partition: 0	Leader: 0	Replicas: 0	Isr: 0
  	Topic: log	Partition: 1	Leader: 0	Replicas: 0	Isr: 0
  ```

- 启动消费端服务进行接收消息

  ```shell
  bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic log
  ```

- 启动生产端服务进行发送消息

  ```shell
  bin/kafka-console-producer.sh --broker-list localhost:9092 --topic log
  ```

  