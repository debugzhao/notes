查看防火墙状态：

```shell
systemctl status firewalld
```

临时关闭防火墙

```shell
systemctl stop firewalld
```

永久关闭防火墙

```shell
systemctl disable firewalld
```

打开防火墙命令

```shell
systemctl enable firewalld
```

#### ES服务配置

**增加Linux系统部署软件的内存和硬盘**

```xml
vim /etc/security/limits.conf
```

在最后一样添加如下配置

```shell
* soft nproc 655350
* soft nofile 655350
* hard nproc 655350
* hard nofile 655350
```

**配置ES最大线程数**

```
vim /etc/sysctl.conf
```

在最后一行添加

```
vm.max_map_count = 262144
```

**配置用户最大的线程数**

```
vim /etc/security/limits.d/90-nproc.conf
```

**使以上所有配置永久生效**

```shell
[root@ES01 opt]# sysctl -p
vm.max_map_count = 262144
```

 **启动ES**

```shell
# 进入bin目录
[root@ES01 bin]# ./elasticsearch
```

第一次启动报错

```java
java.lang.RuntimeException: can not run elasticsearch as root
// 报错原因：因为root用户的权限过大，不能以root用户身份启动ES
```

解决方案：创建用户，给该用户分配在/elasticsearch目录下所有文件的执行权限，然后切换刚才创建的用户，以该用户身份启动ES服务。

启动成功后，如果在虚拟机浏览器可以访问  http://192.168.10.131:9200/，（192.168.10.131是我自己的虚拟机ip地址）但是物理机却不能访问 http://192.168.10.131:9200/的话，一般可以通过关闭虚拟机防火墙或者开放虚拟机90端口就可以解决。（二选一即可，更改为配置后记得**重启虚拟机**）

启动成功后浏览器返回Json字符串

```json
{
  "name" : "node-1",
  "cluster_name" : "my-cluster",
  "cluster_uuid" : "xaAyktXiSU25xWUKoA33CQ",
  "version" : {
    "number" : "7.7.0",
    "build_flavor" : "default",
    "build_type" : "tar",
    "build_hash" : "81a1e9eda8e6183f5237786246f6dced26a10eaf",
    "build_date" : "2020-05-12T02:01:37.602180Z",
    "build_snapshot" : false,
    "lucene_version" : "8.5.1",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```

**配置ES的yml文件**

```yml
# ES集群名称
cluster.name: my-cluster
# ES节点名称，如果是集群模式，每个节点名称不能重复
node.name: node-1
# ES数据文件存放目录
path.data: /opt/ElasticSearch/elasticsearch-7.7.0/data
# ES日志文件存放目录
path.data: /opt/ElasticSearch/elasticsearch-7.7.0/data
# 释放ES内存锁，让ES拥有最大的内存
bootstrap.memory_lock: false
# 配置允许连接的IP地址，这里配置为0.0.0.0，表示所有的终端都可以连接ES服务
network.host: 0.0.0.0
# ES默认端口号为9200，不建议修改
http.port: 9200
# 识别集群中其他节点的host，如果是单节点只需要配置一个host即可
discovery.seed_hosts: ["192.168.10.131"]
```

### IK分词器

下载IK分词器插件，放到你的ES安装目录下的plugins目录中，然后解压，IK分词器配置完成。

**注意事项：**你的IK分词器版本和ES版本必须保持一致，即IK分词器版本不能高于也不能低于ES版本，否则ES服务启动报版本不匹配错误。

博主ES版本为7.7.0，IK分词器版本也必须为7.7.0。

### Kibana

`介绍：`Kibana是一款开源的数据分析和可视化平台（主要用于操作ES），他可以非常方便的对ES进行添加index，type操作，document操作（数据的增删改查）

`Linux配置Kibana：`



### 第1章 Elasticsearch 概述

#### 1.1 Elasticsearch 是什么

The Elastic Stack, 包括 Elasticsearch、Kibana、Beats 和 Logstash（也称为 ELK Stack）。 能够安全可靠地获取任何来源、任何格式的数据，然后实时地对数据进行搜索、分析和可视化。

Elasticsearch是一个开源的高扩展的分布式全文搜索引擎，它可以近乎实时的存储、搜索海量数据，可以扩展到上百台服务器，处理PB级别数据。

#### 1.2 Elasticsearch特点

1. 搜索之外还可以统计分析数据
2. Elasticsearch具有良好得伸缩性，对分布式环境比较友好
3. Elasticsearch具有数据**分析指标**的功能

### 第2章 Elasticsearch 入门

#### 2.1 Elasticsearch 基本操作

##### 2.1.1 数据格式

在Elasticsearch 7.x版本中，Type的概念已经被删除了

<img src="https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/image.1nf59w8yjsrk.png" alt="image" style="zoom:67%;" />

##### 2.2.2 索引

1. 倒排索引

   根据keyword关联记录id，再根据id查询内容，这种索引方式称之为倒排索引

   ```properties
   keyword    id 
   -------------------
   name      101,102
   zhang     101
   san       101
   ```

2. 正向索引

   根据id查询文章内容，再根据文章内容去筛选关键字，这种索引方式称为正排索引

   ```properties
   id    content 
   -------------------
   101   my name is zhang san
   102   my name is li si
   ```

##### 2.2.2 索引操作

Elasticsearch 中的Index可以理解为MySQL中的database

1. 映射关系

   ```json
   {
       "properties": {
           "name": {
               "type": "text", // 文本，可以被分词
               "index": true
           },
           "sex": {
               "type": "keyword", //关键字，不能被分词 
               "index": true
           },
           "age": {
               "type": "long",
               "index": false // 不能被索引
           }
       }
   }
   ```

   



