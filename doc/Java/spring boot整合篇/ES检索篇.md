#### 启动ES

```shell
docker run -e ES_JAVA_OPTS="-Xms256m -Xmx256m" -d -p 9200:9200 -p 9300:9300 --name ES01 5acf0e8da90b
```

#### [中文文档](https://www.elastic.co/guide/cn/elasticsearch/guide/current/getting-started.html)

#### ES架构

<img src="C:\Users\Administrator\Desktop\md\Java\spring boot整合篇\image\ES架构.png" style="zoom: 50%;" />

#### spring boot对ES的支持

> 1、Jest
> 	 默认不生效，需要导入Jest工具包 io.searchBox.client.JestClient
> 2.SpringData ElasticSearch
> 	2.1 client节点信息：clusterNode集群节点、clusterName集群名称
> 	2.2 通过ElasticSearchTemplate操作ES
> 	2.3 编写一个ElasticSearchRepository操作ES

#### JestClient添加数据

```java
/**
* 注入JestClient对ES进行操作
*/
@Autowired
JestClient jestClient;

/**
* 向ES中构建一个索引
*/
@Test
void createESIndex() throws IOException {
    //new 一个对象用于构建索引
    Article article = new Article(1, "好消息", "张三", "hello world");
    //构建一个index索引
    Index builder = new Index.Builder(article).index("atguigu").type("news").build();
    jestClient.execute(builder);
}
```

#### spring boot对spring-data-elasticsearch的支持

##### pom依赖

```xml
<!--引入elasticsearch支持-->
    <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
    <version>2.1.3.RELEASE</version>
</dependency>
```

##### 配置文件

```yaml
spring:
  data:
    elasticsearch: 
      cluster-name: elasticsearch
      cluster-nodes: 192.168.10.131:9300
```

##### repository

```java
package com.spring.springsenior.ElasticSearch.repository;

import com.spring.springsenior.ElasticSearch.entity.Article;
import org.springframework.data.elasticsearch.repository.ElasticsearchRepository;
import org.springframework.stereotype.Repository;

/**
 * @author : Jeffersonnn
 * @date : 2020/2/4
 * @description :
 */
@Repository
public interface ArticleRepository extends ElasticsearchRepository<Article,Integer> {
}

```

##### 实体类Article

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
@Document(indexName = "atguigu",type = "article") //指定索引名称(数据库名)和类别名称(表名)
public class Article {
    @JestId
    private Integer id;
    private String author;
    private String title;
    private String content;
}
```

##### 测试方法

```java
@Autowired
ArticleRepository articleRepository;

/**
* 通过SpringData elasticsearch方式实现新增一条数据
*/
@Test
void index(){
    Article article = new Article(1, "吴承恩", "西游记", "三打白骨精");
    articleRepository.index(article);
}
```

##### 版本不兼容导致报错

```java
//报错信息
[{#transport#-1}{729dgKKVSF-ti27v2_w68g}{127.0.0.1}{127.0.0.7:9300}]]
```

原因：

> 1.spring boot2.X版本必须使用Elasticsearch 5.X版本
>
> 2.Elasticsearch 2.X的版本必须使用spring boot1.5版本
>
> 3.目前spring-boot-starter-data-elasticsearch还不支持Elasticsearch 6.X版本