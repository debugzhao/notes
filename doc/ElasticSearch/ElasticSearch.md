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

##### 2.1.2 索引

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

##### 2.1.3 索引操作

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


##### 2.1.4 Java API操作

Maven依赖

```xml
<dependencies>
    <dependency>
        <groupId>org.elasticsearch</groupId>
        <artifactId>elasticsearch</artifactId>
        <version>7.8.0</version>
    </dependency>
    <!-- elasticsearch 的客户端 -->
    <dependency>
        <groupId>org.elasticsearch.client</groupId>
        <artifactId>elasticsearch-rest-high-level-client</artifactId>
        <version>7.8.0</version>
    </dependency>
    <!-- elasticsearch 依赖 2.x 的 log4j -->
    <dependency>
        <groupId>org.apache.logging.log4j</groupId>
        <artifactId>log4j-api</artifactId>
        <version>2.8.2</version>
    </dependency>
    <dependency>
        <groupId>org.apache.logging.log4j</groupId>
        <artifactId>log4j-core</artifactId>
        <version>2.8.2</version>
    </dependency>
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
        <version>2.9.9</version>
    </dependency>
    <!-- junit 单元测试 -->
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
    </dependency>
</dependencies>
```

ESClientUtil

```java
public class ESClientUtil {

    private static final String hostname = "localhost";

    private static final int port = 9200;

    public static final RestHighLevelClient esClient = new RestHighLevelClient(RestClient.builder(new HttpHost(hostname, port)));

    public static void esClientClose() throws IOException {
        ESClientUtil.esClient.close();
    }
}
```

索引操作

```java
public class ESClientIndexTest {

    public static void main(String[] args) throws IOException {
        // createIndex();
        // queryIndex();
        deleteIndex();
        ESClientUtil.esClientClose();
    }

    /**
     * 删除索引
     * @throws IOException
     */
    private static void deleteIndex() throws IOException {
        DeleteIndexRequest deleteIndexRequest = new DeleteIndexRequest("man");
        AcknowledgedResponse response = ESClientUtil.esClient.indices().delete(deleteIndexRequest, RequestOptions.DEFAULT);
        boolean acknowledged = response.isAcknowledged();
        System.out.println("删除状态：" + acknowledged);
    }

    /**
     * 删除索引
     * @throws IOException
     */
    private static void queryIndex() throws IOException {
        GetIndexRequest indexRequest = new GetIndexRequest("man");
        GetIndexResponse response = ESClientUtil.esClient.indices().get(indexRequest, RequestOptions.DEFAULT);
        System.out.println("索引别名：" + response.getAliases());
        System.out.println("映射关系：" + response.getMappings());
        System.out.println("配置信息：" + response.getSettings());
    }


    /**
     * 创建索引
     * @throws IOException
     */
    public static void createIndex() throws IOException {
        CreateIndexRequest indexRequest = new CreateIndexRequest("finance");
        CreateIndexResponse indexResponse = ESClientUtil.esClient.indices().create(indexRequest, RequestOptions.DEFAULT);
        boolean acknowledged = indexResponse.isAcknowledged();
        System.out.println("创建索引结果：" + acknowledged);
    }
}
```

文档操作

```java
public class ESClientDocTest {
    public static void main(String[] args) throws IOException {
        // insertDoc();
        // updateDoc();
        // insertBatch();
        // deleteBatch();
        queryAll();
        ESClientUtil.esClientClose();
    }


    /**
     * 全量查询
     * @throws IOException
     */
    private static void queryAll() throws IOException {
        SearchRequest request = new SearchRequest();
        request.indices("user");
        // 1.全量查询
        /*request.source(new SearchSourceBuilder().query(QueryBuilders.matchAllQuery()));
        SearchResponse response = ESClientUtil.esClient.search(request, RequestOptions.DEFAULT);*/

        // 2.条件查询
        /*request.source(new SearchSourceBuilder().query(QueryBuilders.termQuery("age", 30)));
        SearchResponse response = ESClientUtil.esClient.search(request, RequestOptions.DEFAULT);*/

        // 3.分页查询
        /*SearchSourceBuilder builder = new SearchSourceBuilder().query(QueryBuilders.matchAllQuery());
        request.source(builder.from(0).size(3));
        SearchResponse response = ESClientUtil.esClient.search(request, RequestOptions.DEFAULT);*/

        // 4.结果排序
        /*SearchSourceBuilder builder = new SearchSourceBuilder().query(QueryBuilders.matchAllQuery());
        request.source(builder.sort("age", SortOrder.DESC));
        SearchResponse response = ESClientUtil.esClient.search(request, RequestOptions.DEFAULT);*/

        // 5.过滤指定字段
        /*SearchSourceBuilder builder = new SearchSourceBuilder().query(QueryBuilders.matchAllQuery());
        String[] includeField = {"age", "name"};
        String[] excludeField = {};
        builder.fetchSource(includeField, excludeField);
        request.source(builder);
        SearchResponse response = ESClientUtil.esClient.search(request, RequestOptions.DEFAULT);*/

        // 6.过滤指定字段
        /*SearchSourceBuilder builder = new SearchSourceBuilder().query(QueryBuilders.matchAllQuery());
        String[] includeField = {"age", "name"};
        String[] excludeField = {};
        builder.fetchSource(includeField, excludeField);
        request.source(builder);
        SearchResponse response = ESClientUtil.esClient.search(request, RequestOptions.DEFAULT);*/

        // 7.条件查询
        /*SearchSourceBuilder builder = new SearchSourceBuilder();
        BoolQueryBuilder boolQueryBuilder = QueryBuilders.boolQuery();
        // boolQueryBuilder.must(QueryBuilders.matchQuery("name", "lucas"));
        // boolQueryBuilder.must(QueryBuilders.matchQuery("sex", "female"));
        boolQueryBuilder.should(QueryBuilders.matchQuery("sex", "male"));
        boolQueryBuilder.should(QueryBuilders.matchQuery("age", 30));
        builder.query(boolQueryBuilder);
        request.source(builder);*/

        // 8.范围查询
        /*SearchSourceBuilder builder = new SearchSourceBuilder();
        BoolQueryBuilder boolQueryBuilder = QueryBuilders.boolQuery();
        boolQueryBuilder.must(QueryBuilders.matchQuery("sex", "male"));
        RangeQueryBuilder ageRangeQuery = QueryBuilders.rangeQuery("age");
        ageRangeQuery.gte(30);
        ageRangeQuery.lte(50);
        boolQueryBuilder.must(ageRangeQuery);
        builder.query(boolQueryBuilder);
        request.source(builder);*/

        // 9.模糊查询
        /*SearchSourceBuilder builder = new SearchSourceBuilder();
        builder.query(QueryBuilders.fuzzyQuery("name", "luc").fuzziness(Fuzziness.TWO));
        request.source(builder); */

        // 10.高亮查询
        /*SearchSourceBuilder builder = new SearchSourceBuilder();

        HighlightBuilder highlightBuilder = new HighlightBuilder();
        highlightBuilder.preTags("<font color='red'>");
        highlightBuilder.postTags("</font>");
        highlightBuilder.field("name");

        builder.highlighter(highlightBuilder);
        builder.query(QueryBuilders.termQuery("name", "lucas"));
        request.source(builder);*/

        // 11.聚合查询
        SearchSourceBuilder builder = new SearchSourceBuilder();
        MaxAggregationBuilder maxAggregationBuilder = AggregationBuilders.max("maxAge").field("age");
        builder.aggregation(maxAggregationBuilder);
        request.source(builder);

        SearchResponse response = ESClientUtil.esClient.search(request, RequestOptions.DEFAULT);
        System.out.println(response);
        /*SearchHits hits = response.getHits();
        System.out.println("查询消耗时间：" + response.getTook());
        System.out.println("查询命中条数：" + hits.getTotalHits());
        for (SearchHit hit : hits.getHits()) {
            System.out.println(hit.getSourceAsString());
        }*/
    }

    /**
     * 批量删除
     * @throws IOException
     */
    private static void deleteBatch() throws IOException {
        BulkRequest bulkRequest = new BulkRequest();
        bulkRequest.add(new DeleteRequest().index("user").id("201"));
        bulkRequest.add(new DeleteRequest().index("user").id("202"));
        bulkRequest.add(new DeleteRequest().index("user").id("203"));
        BulkResponse bulkResponse = ESClientUtil.esClient.bulk(bulkRequest, RequestOptions.DEFAULT);
        System.out.println(bulkResponse.getTook());
        System.out.println(Arrays.toString(bulkResponse.getItems()));
    }

    /**
     * 批量添加文档
     * @throws IOException
     */
    private static void insertBatch() throws IOException {
        BulkRequest bulkRequest = new BulkRequest();
        String jsonString1 = JSONObject.toJSONString(new User("natasha", 24, "female"));
        bulkRequest.add(new IndexRequest().index("user").id("201").source(jsonString1, XContentType.JSON));
        String jsonString2 = JSONObject.toJSONString(new User("andre", 24, "male"));
        bulkRequest.add(new IndexRequest().index("user").id("202").source(jsonString2, XContentType.JSON));
        String jsonString3 = JSONObject.toJSONString(new User("王冰冰", 30, "female"));
        bulkRequest.add(new IndexRequest().index("user").id("203").source(jsonString3, XContentType.JSON));
        String jsonString4 = JSONObject.toJSONString(new User("王健林", 46, "male"));
        bulkRequest.add(new IndexRequest().index("user").id("204").source(jsonString4, XContentType.JSON));
        String jsonString5 = JSONObject.toJSONString(new User("jack", 41, "male"));
        bulkRequest.add(new IndexRequest().index("user").id("205").source(jsonString5, XContentType.JSON));
        String jsonString6 = JSONObject.toJSONString(new User("mask", 45, "female"));
        bulkRequest.add(new IndexRequest().index("user").id("206").source(jsonString6, XContentType.JSON));

        BulkResponse bulkResponse = ESClientUtil.esClient.bulk(bulkRequest, RequestOptions.DEFAULT);
        System.out.println(bulkResponse.getTook());
        System.out.println(Arrays.toString(bulkResponse.getItems()));
    }

    /**
     * 更新文档
     * @throws IOException
     */
    private static void updateDoc() throws IOException {
        UpdateRequest updateRequest = new UpdateRequest();
        updateRequest.index("user").id("101");
        updateRequest.doc(XContentType.JSON, "sex", "female");
        UpdateResponse updateResponse = ESClientUtil.esClient.update(updateRequest, RequestOptions.DEFAULT);
        System.out.println(updateResponse.getResult());
    }

    /**
     * 插入文档
     * @throws IOException
     */
    private static void insertDoc() throws IOException {
        IndexRequest indexRequest = new IndexRequest();
        indexRequest.index("user").id("101");
        User user = new User("lucas", 24, "male");
        String jsonString = JSONObject.toJSONString(user);
        indexRequest.source(jsonString, XContentType.JSON);
        IndexResponse response = ESClientUtil.esClient.index(indexRequest, RequestOptions.DEFAULT);
        DocWriteResponse.Result result = response.getResult();
        System.out.println(result);
    }
}
```

### 第3章 Elasticsearch 环境

#### 3.1 相关概念

##### 3.1.1 单机 & 集群

单台 Elasticsearch 服务器提供服务，往往都有最大的负载能力，超过这个阈值，服务器 性能就会大大降低甚至不可用，所以生产环境中，一般都是运行在指定服务器集群中。

除了负载能力，单点服务器也存在其他问题：

1. 单台服务器存储容量有限
2. 单台服务器出现故障时，无法实现高可用
3. 单台服务器的并发能力有限

##### 3.1.2 集群 Cluster

一个集群就是由两台服务器节点组织在一起，共同持有整个的数据，并一起提供存储和搜索功能。一个 Elasticsearch 集群有一个唯一的名字标识，这个名字默认就 是`elasticsearch`。这个名字是重要的，因为一个节点只能通过指定某个集群的名字，来加入 这个集群。

### 第4章 Elasticsearch 进阶

#### 4.1 核心概念

##### 4.1.1 索引（Index）

Elasticsearch 索引的精髓：一切设计都是为了提高搜索的性能。

##### 4.1.3 字段（Field）

相当于是数据表的字段，对文档数据根据不同属性进行的分类标识。

##### 4.1.5 映射（Mapping）

mapping 是处理数据的方式和规则方面做一些限制，如：某个字段的数据类型、默认值、 分析器、是否被索引等等。

这些都是映射里面可以设置的，其它就是处理 ES 里面数据的一 些使用规则设置也叫做映射，**按着最优规则处理数据对性能提高很大，**因此才需要建立映射， 并且需要思考如何建立映射才能对性能更好。

##### 4.1.6 分片（Shards）

> 在MySQL数据库中可以理解为对一个数据库按照表进行拆分

一个索引可以存储超出单个节点硬件限制的大量数据。比如，一个具有 10 亿文档数据 的索引占据 1TB 的磁盘空间，而任一节点都可能没有这样大的磁盘空间。或者单个节点处 理搜索请求，响应太慢。为了解决这个问题，Elasticsearch 提供了将索引划分成多份的能力， 每一份就称之为分片。当你创建一个索引的时候，你可以指定你想要的分片的数量。每个分 片本身也是一个功能完善并且独立的“索引”，**这个“索引”可以被放置到集群中的任何节点 上。**

分片很重要，主要有两方面的原因：

1. 允许你水平分割 / 扩展你的内容容量
2. 允许你在分片之上进行分布式的、并行的操作，进而提高性能/吞吐量

至于一个分片怎样分布，它的文档怎样聚合和搜索请求，是完全由 Elasticsearch 管理的， 对于作为用户的你来说，这些都是透明的，无需过分关心。

**被混淆的概念是，一个 Lucene 索引 我们在 Elasticsearch 称作 分片 。 一个 Elasticsearch 索引 是分片的集合。 当 Elasticsearch 在索引中搜索的时候， 他发送查询 到每一个属于索引的分片(Lucene 索引)，然后合并每个分片的结果到一个全局的结果集。**

##### 4.1.7 副本（Replicas）

在一个网络 / 云的环境里，失败随时都可能发生，在某个分片/节点不知怎么的就处于 离线状态，或者由于任何原因消失了，这种情况下，有一个故障转移机制是非常有用并且是 强烈推荐的。为此目的，Elasticsearch 允许你创建分片的一份或多份拷贝，这些拷贝叫做复 制分片(副本)。

复制分片之所以重要，有两个主要原因：

1. 在分片/节点失败的情况下，提供了高可用性。因为这个原因，注意到复制分片从不与 原/主要（original/primary）分片置于同一节点上是非常重要的。
2. 扩展你的搜索量/吞吐量，因为搜索可以在所有的副本上并行运行。

##### 4.1.8 分配（Allocation）

将分片分配给某个节点的过程，包括分配主分片或者副本。如果是副本，还包含从主分 片复制数据的过程。这个过程是由 master 节点完成的。

#### 4.2 系统架构

![image](https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/image.6yb59m4cwfw0.png)

当有节点加入群集或者从集群中移除时，集群会重新分布所有的数据

当一个节点选举称为主节点时，他将负责集群范围内的所有的变更，例如增加、删除索引，增加、删除节点；此外主节点不涉及文档级别的变更和搜索操作，所以如果集群中只有一个主节点的话，增加搜索操作主节点也不会出现性能瓶颈。

作为用户，我们可以将请求发送给集群中的任意节点，包括主节点。每个节点都知道任意文档所存储的位置，并且可以直接将请求转发给我们需要查询的文档所在的节点。无论我们把请求发送给哪个节点，她都可以负责从各个包含所需文档的节点里面收集回数据，并且返回给客户端。

#### 4.3分布式集群

##### 4.3.1单节点集群

整个集群中只有一个节点

创建一个分片数为3，副本数为1的peaple索引

<img src="C:\Users\lucas.zhao\AppData\Roaming\Typora\typora-user-images\image-20210815130743939.png" alt="image-20210815130743939" style="zoom: 80%;" />

```json
# 127.0.0.1:9201/peaple

{
    "settings": {
        "number_of_shards": 3,
        "number_of_replicas": 1
    }
}
```

##### 4.3.2 故障转移

当集群中只有一个节点在运行时，意味着会有一个单点故障问题——没有冗余

![image](https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/image.jrm5k0mylvk.png)

##### 4.3.3 水平扩容

<img src="https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/image.5zcz1cqc0sk0.png" alt="image" style="zoom:80%;" />

Node 1 和 Node 2 上各有一个分片被迁移到了新的 Node 3 节点，现在每个节点上都拥有 2 个分片， 而不是之前的 3 个。 这表示每个节点的硬件资源（CPU, RAM, I/O）将被更少的分片所共享，每个分片 的性能将会得到提升。

##### 4.3.4 应对故障

##### 4.3.5 路由计算

ElasticSearch是如何知道一个文档应该存放在哪个分片位置上呢？实际上计算分片位置遵循路由计算公式：

```shell
# shard_index 分片位置
# hash 哈希算法
# routing是一个可变值，默认是文档id
# number_of_primary_shards 主分片的数量
shard_index = hash(routing) % number_of_primary_shards
```

这也就解释了为什么创建索引的时候要确定主分片的数量，并且以后永远也不会再改变这个数量。因为一旦索引的主分片的数量发生改变，之前文档的索引值将会失效，文档也就再也索引不到了。

所有的文档API（get、index、delete、bulk、update以及mget）都接收一个叫routing的参数，通过这个参数我们可以自定义文档到分片的映射位置。通过自定义为路由参数我们可以确保相关文档都可以存储到一个文档中。

4.3.5 **写操作**

新建、索引和删除操作都是写操作，必须等主分片完成之后再同步到相关的副分片上，最后才认为写操作成功

**写操作流程**

<img src="https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/image.47x02vto9280.png" alt="image" style="zoom:80%;" />

1. 客户端请求任意的集群节点，此时的节点称之为协调节点
2. 协调节点获取文档的id，通过路由计算，将请求转发至执行的节点上（文档的主分片所在的节点上）
3. 主分片保存数据
4. 主分片保存数据成功之后将数据同步至其他副本（同步复制/异步复制）
5. 副本保存成功之后，向主分片发送反馈消息
6. 主分片获取到反馈消息，向客户端发送反馈消息
7. 客户端接收到反馈消息，写请求结束

**ElasticSearch关于写请求的参数**

| 参数        | 含义                                                         |
| ----------- | ------------------------------------------------------------ |
| consistency | 写请求的一致性参数<br/>one：只要主分片写数据完毕，就认为写操作成功<br/>all：主分片和所有的副本写数据完毕，才认为写操作成功<br/>quorum：默认参数，大多数副本写数据完毕，才认为写操作成功 |
| timeout     | 如果没有足够的副本分片数量，ES会处于等待状态，希望更多的副本分片数量出现。<br/>ES默认等待时间为1分钟，我们也可以手动设置timeout等待时间 |

##### 4.3.6 读操作

<img src="https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/image.3xysg2jory00.png" alt="image" style="zoom:67%;" />

1. 客户端发送查询请求至协调节点
2. 协调节点通过文档id，计算主分片所在的节点以及全部副本分片的位置
3. 为了能够实现负载均衡，减小主分片所在节点的开销，可以轮询所有节点
4. 将请求转发至某一个副本分片所在的节点
5. 节点返回查询结果，将查询结果反馈给客户端



