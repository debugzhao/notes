## 第06章 索引的数据结构

### 1.为什么使用索引

使用索引可以减少磁盘IO，提高查询效率。

### 2.索引及其优缺点

#### 2.1 索引概述

索引（Index）是帮助MySQL高效获取数据的数据结构。

#### 2.2 优点

1. 提高数据检索效率，降低数据库IO成本
2. 通过建立唯一索引， 可以保证数据库中每一行数据的唯一性
3. 可以加速表与表之间的连接

#### 2.3 缺点

1. 创建和维护索引需要消耗时间，并且随着数据量的增加，消耗的时间也会增加
2. 索引本身也会占用磁盘空间
3. 虽然索引会大大提高查询效率，但是同时也会降低更新表的效率

### 3.InnoDB中索引的推演

#### 3.1 索引之前的查找

#### 3.2 设计索引

##### 一个简单的索引设计方案

为了快速定位数据，我们需要建立一个目录，建立这个目录必须符合以下特点：

1. 下一个数据页中的数据的主键必须大于上一个数据页中的数据主键
2. 给所有也简历一个目录项

![image](https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/image.43n3prsata20.webp)

这个目录还有一个别名就是**索引**。

##### InnoDB中的索引方案

1. 第一次迭代：记录目录项的页

   ![image](https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/image.4c5iu2f89lu0.webp)

2. 第二次迭代：多个记录目录项的页

   ![image](https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/image.143w4l7h9hj4.webp)

3. 第三次迭代：目录页的目录页

   只要目录页超过两个，需要新键一个更高层级的目录页

   ![image](https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/image.5zqgo4vq6pk0.webp)

4. B+ Tree

   ![image](https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/image.7k02bo7d89k0.webp)

   以上数据结构就是B树

   假设`叶子节点`所代表的数据页可以存放`100条用户数据`，非`叶子节点`所代表的的数据页可以存放`1000条目录数据`，那么：

   - 如果B+树只有<font color="red">1层</font>，也就是只有1个用于存放用户记录的节点，最多能存放 <font color="red">100 条记录</font>
   - 如果B+树有<font color="red">2层</font>，最多能存放<font color="red"> 1000×100=10,0000 条记录</font>
   - 如果B+树有<font color="red">3层</font>，最多能存放 <font color="red">1000×1000×100=1,0000,0000 条记录</font>
   - 如果B+树有<font color="red">4层</font>，最多能存放 <font color="red">1000×1000×1000×100=1000,0000,0000 条记录</font>

   <font color="red">**所以一般情况下我们用到的B+树都不会超过4层。**</font>

#### 3.3 常见索引概念

索引按照物理实现方式，索引可以分为 2 种：聚簇（聚集）和非聚簇（非聚集）索引。我们也把非聚集 索引称为二级索引或者辅助索引。

##### 聚簇索引

1. 概念

   由主键构成的索引称为聚簇索引

2. 特点

   页内的记录是按照主键的大小顺序组成一个<font color="red">单项链表</font>

   页之间按照主键大小组成一个<font color="red">双向链表</font>
   <font color="red">B+树的叶子节点存储的是完整的用户数据</font>

3. 优点

   - 因为聚簇索引将索引和数据保存在同一个B+树中，所以从聚簇索引中查找数据比非聚簇索引中<font color="red">访问数据更快</font>
   - 聚簇索引对主键的排序查找和范围查找速度更快
   - 节省了大量的IO操作

4. 缺点

   - 插入速度严重依赖于插入顺序，因此我们一般会定义一个自增的ID作为主键
   - 更新主键的代价很高
   - <font color="red">访问二级索引需要进行两次索引查找，第一次找到主键值，第二次根据主键值找到行数据</font>
   
5. 限制

   - 对于MySQL数据库目前<font color="red">只有InnoDB引擎支持聚簇索引</font>，MyISAM不支持聚簇索引

   - 由于数据物理存储方式只能有一种，所以每个<font color="red">MySQL表只能有一个聚簇索引</font>。一般情况下是该表的主键

   - 如果没有定义主键，InnoDB会选择<font color="red">非空的唯一索引代替</font>。如果没有这样的索引，InnoDB会隐式定义一个主键来作为聚簇索引。

   - 为了充分利用聚簇索引聚簇的特性，索引InnoDB表主键尽量<font color="red">选择有序的顺序id</font>，不要选择无序的id，比如UUID，MD5，HASH、字符串列作为主键而无法保证数据的顺序增长。

     > 雪花算法生成的主键也是按照时间增长的。 

##### 二级索引

> 二级索引也称为辅助索引、非聚簇索引
>
> 一个数据库表中只能有一个聚簇索引，但是可以有多个非聚簇索引，即多个二级索引

1. 概念

   <font color="red">由非主键列创建的索引称为非聚簇索引</font>，即二级索引。<font color="red">二级索引的叶子节点只存储了索引列和主键列</font>，所以通过叶子节点可以观察出是否是二级索引。

2. 回表

   我们根据C2列关键字在二级索引B+树中查询到主键值，然后再根据主键值在聚簇索引B+树中查询其他字段，这种行为需要查询两次B+树的行为就成为回表

3. 问题

   **Q:** 为什么我们需要进行一次回表操作呢？直接把完整的用户记录放到二级索引的叶子结点中不行吗？

   **A：**如果表中的字段非常多的话这将造成非常大的冗余

##### 联合索引

同时为多个列建立索引称为联合索引，联合索引本质上也是二级索引。

![image](https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/image.4qk4sigaaiw0.webp)

#### 3.4 InnoDB的B+树索引的注意事项

##### 根页面位置万年不动

##### 内节点中目录项记录的唯一性

##### 一个页面最少存储两条记录

### 4.MyISAM的索引方案

#### 4.1 MyISAM索引的原理

#### 4.2 MyISAM和InnoDB对比

MyISAM的索引方式都是“非聚簇”的，与InnoDB包含1个聚簇索引是不同的。

### 5.索引的代价

1. 空间上的代价

   每建立一个索引就会为它创建一个B+树。每一个B+树的每一个节点都是一个数据页，一个数据页的默认占用16KB存储空间，一棵很大的B+树由许多数据页组成，那就是很大的一片存储空间。

2. 时间上的代价

   每次对表中的数据进行增、删、改操作都会去修改各个B+树的索引。如果我们创建了太多不必要的索引，反而会降低表的维护性能。

### 6.MySQL数据结构选择的合理性

#### 6.1 全表遍历

#### 6.2 Hash结构

![](https://s3.bmp.ovh/imgs/2022/05/17/a636ddf0b174e360.png)

哈希函数h有可能将两个不同的关键字映射到相同的位置，这叫做<font color="red">哈希碰撞</font>，在数据库中一般采用链接法 来解决。在链接法中，<font color="red">将散列到同一槽位的元素放在一个链表中。</font>

**Hash索引适用的存储引擎**

| 索引/存储引擎 | MyISAM | InnoDB | Memory                        |
| ------------- | ------ | ------ | ----------------------------- |
| HASH索引      | 不支持 | 不支持 | <font color="red">支持</font> |

**自适应Hash**

![](https://s3.bmp.ovh/imgs/2022/05/17/1766e3b0b1b951e9.png)

采用自适应 Hash 索引目的是方便根据 SQL 的查询条件加速定位到叶子节点，特别是当 B+ 树比较深的时 候，通过自适应 Hash 索引可以明显提高数据的检索效率。

#### 6.3 二叉搜索树

如果利用二叉树作为索引数据结构，那么磁盘的IO次数和索引树的高度是相关的。

1. 二叉搜索树的特点

   ![](https://i.bmp.ovh/imgs/2022/05/17/dcf60450345eb4c3.png)

2. 查找规则

   ![](https://i.bmp.ovh/imgs/2022/05/17/70474395173bfb79.png)

   ![](https://i.bmp.ovh/imgs/2022/05/17/2d5f9c57872052a7.png)

   上面这颗树也属于二分查找树，但是性能上已经退化成了链表的性能，所以查询数据的<font color="red">复杂度是O(n)</font>。为了提高查询效率，就需要<font color="red">减少磁盘IO</font>。为了减少磁盘IO就需要尽量<font color="red">降低树的高度</font>，需要把原来瘦高的树变成矮胖的树，即<font color="red">树每层的分叉越多越好</font>。

#### 6.4 AVL树

为了解决在特殊情况下二叉查找树退化成链表的问题，人们提出了<font color="red">平衡二叉树</font>，也称为AVL树。AVL树具有以下性质：<font color="red">它是一颗空树或者左右两个子树的高度差的绝对值不超过1，并且左右两个子树都是一个平衡二叉树。</font>

常见的平衡二叉树有：平衡二叉搜索胡、红黑树、树堆、伸展树，AVL树的查询的时间复杂度为O(log2n)

![image](https://raw.githubusercontent.com/Andre235/-community/master/src/image.1pkqidicy5j4.webp)

#### 6.5 B-Tree

B树英文称为Balance Tree，也就是多路平衡查找树。B树的高度远小于平衡二叉树的高度。B树作为多路平衡查找树，它的每一个节点最多可以包括M个子节点，<font color="red">M称为B树的阶</font>。每个磁盘块包含关键字和子节点的指针。

![image](https://raw.githubusercontent.com/Andre235/-community/master/src/image.15gc4e3acdek.webp)

B树区别于B+树，它的非叶子节点也是会存储数据的。

**小节**

1. B树在插入和删除的过程中可能会导致树的不平衡，此时需要调整节点的位置来保证树的自平衡
2. B树的关键字分布在整个树中，即叶子节点和非叶子节点都会存储数据。搜索有可能会在非叶子节点中结束
3. B树的搜索性能相当于在关键字全集内做了一次二分查找 

#### 6.6 B+Tree

B+树也是一种<font color="red">多路搜索树，基于B树做出了改进</font>，主流的DBMS都支持B+树的索引方式，比如MySQL。相比较于B树，<font color="red">B+树更适合与文件索引系统</font>。

**B+树和B树的区别在于**

1. 在B+树中，有K个关键字就有K个孩子节点，也就是孩子节点的数量 = 关键字的数量；在B树中，孩子节点的数量 = 关键字数量 + 1
2. 在B+树中，叶子节点的关键字也会出现在非叶子节点中，并且所有的子节点的关键字最大（或者最小）
3. 在B+树中，非叶子节点的关键字仅用于索引，不保存数据记录，跟记录有关的信息都存储在叶子节点中；但是在B树中，<font color="red">非叶子节点即用于索引又用户存储用户数据。</font>
4. 在B+树中所有的关键字都在叶子节点中出现，并且叶子结点构成了一个有序链表。而叶子节点本身是按照关键字大小有序连接的。

B+树的非叶子节点不直接存储数据，这样设计的好处在于：

1. <font color="red">B+树查询效率更稳定；</font>
2. <font color="red">B+树查询效率更高；</font>B+树查询效率更高，因为B+树存储的是索引，在数据页容量固定的情况下B+树可以存放更多的数据索引，也就是会有更多的关键字（跟多的分叉| 更多的子节点），从而树型结构更加矮胖，查询效率相对于B树更高；
3. <font color="red">在范围查询上B+树得查询效率也比B树更高。</font>这是因为所有的关键字都会出现在B+树的叶子节点中，并且叶子节点又是一个有序的链表，我们通过指针的方式可以更高效地进行范围查找。但是在B树中需要通过中序遍历才能够实现范围查找， 效率要低很多。

**面试题：**

1. 为了减少IO，索引树会一次性加载吗？

   数据库所以是存储在磁盘上的，如果数据量非常大的话那么索引也会很大，有时候甚至会超过几个G，所以是不可能会直接将索引都加载进内存的，这样会对极大消耗CPU资源。通常的做法是：逐一加载索引树的磁盘页，因为每个磁盘页对应着B+树的节点

2. B+树的存储能力如何？为何说一般查找行记录，最多只需要1-3次IO?

   <font color="red">InnoDB存储引擎的页的大小默认为16KB</font>，一般表的主键类型为INT类型（占4字节），也就是一个页大概可以存储1k个键值（10 ^ 3个键值）。也就是说数据<font color="red">深度为3的B+树中可以维护10 ^ 3 * 10 ^ 3 * 10 ^ 3 = 10亿条记录</font>。

   实际上每个节点可能不能被填充满，因此在<font color="red">数据库中B+树的高度一般在2-4层</font>。MySQL的InnoDB存储引擎在设计时将根节点的信息常驻内存，也就是说查找某一个键值的行记录是最多需要 1- 3次磁盘的IO操作。

3. 为什么说B+树比B树更适合做实际应用中的文件索引和数据库索引？

   由于B+树中非叶子节点存储的是子节点的索引，不直接存储数据，所以一个节点的能够容纳的关键字的数量也就更过，一次性读入内存的关键字也就更多，相对来说磁盘的IO次数就降低了。

   B+树的查询效率更加稳定																																			`

4. Hash索引和B+树索引的区别？

   - Hash索引不能进行范围查询（Hash索引指向的数据是无需的），但是B+树支持范围查询（B+树中叶子节点是一个有序的链表）
   - Hash索引不支持联合索引的最左匹配原则
   - Hash索引不支持排序查询（ORDER BY），也无法支持模糊匹配

5. Hash索引和B+树索引是在键索引的时候手动指定的吗

   针对InnoDB和MyISAM存储引擎默认实用的是B+树索引，无法使用Hash索引。InnoDB存储引擎提供的自适应Hash索引也无需手动指定

#### 6.7 R树

<font color="red">R树在MySQL很少使用，仅支持geometry数据类型。解决了在地图业务中高维空间搜索的问题。</font>

## 第08章 索引的创建和设计原则

### 1.索引的声明和使用

#### 1.1索引的分类

MySQL的索引包括普通索引、唯一性索引、全文索引、单列索引、多列索引和空间索引等

- 从功能逻辑上划分

  普通索引、唯一索引、主键索引、全文索引

- 从物理实现方式划分

  聚簇索引、非聚簇索引

- 从作用字段个数划分

  单列索引和联合索引

#### 1.2创建索引

```mysql
alter table table_name add index index_name(column_name);
```

#### 1.3删除索引

```mysql
alter table table_name drop index index_name;
drop index index_name on table_name;
```

### 2.MySQL8.0索引新特性

#### 2.1支持降序索引

#### 2.2隐藏索引

### 3.索引的设计原则

#### 3.1数据的准备

#### 3.2哪些情况下适合创建索引

##### 字段的数值有唯一性的限制

索引本身可以起到约束的作用，比如说主键索引、唯一索引都可以起到唯一约束的作用。因此在数据库表中如果某个字段是唯一的，就可以直接创建主键索引或者唯一索引，这样可以通过索引更快的检索到某条记录。

##### 频繁作为WHERE查询条件的字段

##### 经常GROUP BY和ORDER BY的列

索引就是让数据按照某种顺序进行存储或者检索，因此经常使用GROUP BY 或者ORDER BY的方式让数据变得有序，从而提高查询效率。如果分组或者排序的字段有多个可以创建联合索引。

##### UPDATE、DELETE的WHERE条件列

##### DISTINCT字段需要创建索引

##### 多表JOIN连接时创建索引注意事项

首先<font color="red">连接表的数量不要超过3张</font>，因为每增加一个表就相当于增加一层循环嵌套，数据量大的话是非常影响查询效率的。其次<font color="red">可以对WHERE条件列添加索引</font>；最后可以对<font color="red">多表的连接的字段添加索引。</font>

##### 使用列类型小的创建索引

##### 使用字符串前缀创建索引

如果我们对一个非常长的字符串创建索引，那么对应的B+树会出现以下问题

1. B+树需要把完整的列存储起来，字符串越长索引占用的内存空间也就越大
2. B+树在做比较时消耗的时间也就越长

但是我们可以截取字符串前面的一部分创建索引，这种索引的创建方式叫做<font color="red">前缀索引</font>，前缀索引虽然不能精确定位记录的位置，但是能定位到相应前缀的位置，然后通过回表的方式定位到记录的位置。这种操作既<font color="red">节约空间</font>又<font color="red">减少了字符串比较的时间</font>

<font color="red">前缀索引带来的问题</font>

如果使用了前缀索引，比如说把address列的前12个字符放到了二级索引列中，那么分组 limit查询将会出现问题

```mysql
SELECT * FROM shop
ORDER BY address
LIMIT 12;
```

因为二级索引不包含完整的address信息，所以无法对前面前12个字符相同，后面字符不同的列进行正确的排序，也就是<font color="red">使用前缀索引的方式无法使用索引排序</font>，不过可以通过文件排序的方式解决该问题。

<font color="red">拓展</font>

> Alibaba《Java开发手册》
>
> 【强制】 在varchar字段建立索引时，必须指定索引的长度，没有必要对全字段简历索引

##### 区分度高（散列度高）的列适合创建索引

列的基数指的是一列中不重复数据的个数。在记录行数一定的情况下列的基数越大，该列的的值就约分散；列的基数越小，该列的值就越集中。

最好为列基数大的列建立索引。

<font color="red">拓展</font>

> 建立联合索引时需要把散列度高的列放在前面

##### 使用最频繁的列放到联合索引的左侧

由于最左匹配原则可以增加联合索引的使用率

##### 在多个字段都要创建索引的情况下联合索引优先于单值索引

#### 3.3限制索引的数目

建议单表创建索引的数量不要超过6个，因为：

1. 每个索引都会占用磁盘空间，索引越多占用的磁盘空间越大
2. 索引会降低UPDATE DELETE INSERT语句的性能
3. 索引越多，会增加MySQL优化器生成执行计划的时间，会降低查询性能

1. 

#### 3.4 哪些情况下不适合创建索引

##### 在where中使用不到的字段不要使用索引

##### 数据量小的表最好不要使用索引

##### 有大量重复数据的列最好不要加索引

##### 避免对经常更新的表创建过多的索引

##### 不建议用无序的值创建索引

##### 删除不再使用或者很少使用的索引

##### 不要定义冗余或者重复的索引

## 第09章 性能分析工具的使用

### 1.数据库服务器的优化步骤

当我们遇到数据库调优问题的时候，该如何思考呢？这里把思考的流程整理成下面这张图。整个流程划分成了 `观察（Show status）` 和 `行动（Action）` 两个部分。字母 S 的部分代表观察（会使 用相应的分析工具），字母 A 代表的部分是行动（对应分析可以采取的行动）。

![](https://i.bmp.ovh/imgs/2022/05/20/c136cbfda9c8c952.png)

![](https://i.bmp.ovh/imgs/2022/05/20/9ac91aed58b28536.png)

### 2.查询系统性能参数

在MySQL中，可以使用 SHOW STATUS 语句查询一些MySQL数据库服务器的 性能参数 、 执行频率 。

```mysql
SHOW [GLOBAL|SESSION] STATUS LIKE '参数'
```

一些常用的性能参数如下：

- Connections：连接MySQL服务器的次数
- Uptime：MySQL服务器的上 线时间
- Slow_queries：慢查询的次数
- Innodb_rows_read：Select查询返回的行数
- Innodb_rows_inserted：执行INSERT操作插入的行数
- Inodb_rows_updated：执行UPDATE操作更新的 行数
- Innodb_rows_deleted：执行DELETE操作删除的行数
- Com_select：查询操作的次数
- Com_insert：插入操作的次数
- Com_delete：删除操作的次数

### 3.统计SQL的查询成本	

一个SQL查询语句在执行查询之前会需要确定查询执行计划，如果存在多个查询执行计划的话，MySQL会计算每个执行计划的成本，从中选择<font color="red">成本最小</font>的作为最终的执行计划。

如果我们想要查询某条SQL的查询成本，可以在执行完这条SQL语句之后在当前回话中查看`last_query_cost`来确定查询成本。它通常也是我们<font color="red">评价一个SQL的查询效率的标准</font>。这个查询成本返回的<font color="red">结果是SQL语句所需要读取的页的数量</font>

```mysql
show status like "last_query_cost";
```

**应用场景**

`show status like "last_query_cost";` 对于比较查询开销是非常有用的，SQL查询时一个动态的过程，从页的加载的角度来看，我们可以得到一下结论

1. 查询数据的位置决定效率

   如果数据页在缓冲区中，那么查询效率是最高的。否则每次还要从内存中或者磁盘中读取，开销会大大增加

2. 查询方式决定效率

   如果我们从磁盘中的对单一页进行随机读，那么效率是比较低的；顺序读的方式的则效率会高很多

### 4.定位执行慢的SQL：查询日志

MySQL慢查询语句是用来记录响应时间超过阈值的SQL语句，如果SQL的运行时间超过`long_query_time`配置的阈值，则会被记录到慢查询日志中，`long_query_time`的默认阈值是10s。

默认情况下MySQL<font color="red">没有开启慢查询日志</font>，需要我们手动配置这个参数。如果<font color="red">不是调优需要的话，不建议开启慢查询日志记录，</font>因为多少会对性能带来一定的影响。

慢查询日志相关的参数：

```mysql
# 查看慢查询日志参数
show variables like "%slow_query_log%";
# 开启慢查询日志
set global slow_query_log = "on";
# 查看慢查询阈值
show variables like "%long_query_time%";
# 设置当前会话慢查询阈值
set long_query_time = 1;
# 查看当前慢查询sql的个数
show status like "slow_queries";
```

#### 开启慢查询日志参数	

1. 查看慢查询日志参数

   ```mysql
   mysql> show variables like "%slow_query_log";
   --------------
   show variables like "%slow_query_log"
   --------------
   
   +----------------+-------+
   | Variable_name  | Value |
   +----------------+-------+
   | slow_query_log | OFF   |
   +----------------+-------+
   1 row in set (0.05 sec)
   ```

2. 开启慢查询日志开关

   ```mysql
   mysql> set global slow_query_log = "on";
   --------------
   set global slow_query_log = "on"
   --------------
   
   Query OK, 0 rows affected (0.01 sec)
   ```

   ```mysql
   mysql> show variables like "%slow_query_log";
   --------------
   show variables like "%slow_query_log"
   --------------
   
   +----------------+-------+
   | Variable_name  | Value |
   +----------------+-------+
   | slow_query_log | ON    |
   +----------------+-------+
   1 row in set (0.00 sec)
   ```

3. 查看慢查询日志文件绝对路径

   ```mysql
   mysql> show variables like "%slow_query_log%";
   --------------
   show variables like "%slow_query_log%"
   --------------
   
   +---------------------+-----------------------------------+
   | Variable_name       | Value                             |
   +---------------------+-----------------------------------+
   | slow_query_log      | ON                                |
   | slow_query_log_file | /home/mysql/data/testttt-slow.log |
   +---------------------+-----------------------------------+
   2 rows in set (0.00 sec)
   ```

4. 修改long_query_time阈值

   ```mysql
   # 设置全局慢查询阈值
   set global long_query_time = 1;
   # 设置当前会话慢查询阈值
   set long_query_time = 1;
   ```

5. MySQL配置文件中一并设置参数

   ```mysql
   # 修改my.cnf文件，可以看做是永久配置参数方式
   [mysqld]
   slow_query_log=on
   slow_query_log_file=/var/lib/mysql/node-slow.log
   long_query_time=3
   log_output=FILE
   ```

#### 查看慢查询数目

```mysql
# 查看慢查询数目，结果表明有三条慢查询SQL
mysql> show status like "slow_queries";
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| Slow_queries  | 3     |
+---------------+-------+
1 row in set (0.00 sec)
```

#### 案例演示

#### 测试及分析

```mysql
# 查看慢查询数目，结果表明有三条慢查询SQL
mysql> show status like "slow_queries";
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| Slow_queries  | 3     |
+---------------+-------+
1 row in set (0.00 sec)
```

#### 慢查询日志分析工具：mysqldumpslow

```shell
# 查看如何使用mysqldumpslow工具
[root@testttt /]# mysqldumpslow --help
Usage: mysqldumpslow [ OPTS... ] [ LOGS... ]

Parse and summarize the MySQL slow query log. Options are

  --verbose    verbose
  --debug      debug
  --help       write this text to standard output

  -v           verbose
  -d           debug
  -s ORDER     what to sort by (al, at, ar, c, l, r, t), 'at' is default
                al: average lock time
                ar: average rows sent
                at: average query time
                 c: count
                 l: lock time
                 r: rows sent
                 t: query time
  -r           reverse the sort order (largest last instead of first)
  -t NUM       just show the top n queries
  -a           don't abstract all numbers to N and strings to 'S'
  -n NUM       abstract numbers with at least n digits within names
  -g PATTERN   grep: only consider stmts that include this string
  -h HOSTNAME  hostname of db server for *-slow.log filename (can be wildcard),
               default is '*', i.e. match all
  -i NAME      name of server instance (if using mysql.server startup script)
  -l           don't subtract lock time from total time
```

```mysql
# 使用mysqldumpslow工具分析慢查询日志
# -s t表示按照query time排序
# -a 不使用抽象的数据类型去代替具体的查询参数
# -t 5 仅展示前几条慢查询SQL
[root@testttt /]# mysqldumpslow -s t -a -t 5 /home/mysql/data/testttt-slow.log

Reading mysql slow query log from /home/mysql/data/testttt-slow.log
Count: 1  Time=0.00s (0s)  Lock=0.00s (0s)  Rows=0.0 (0), 0users@0hosts
  # User@Host: skip-grants user[root] @ localhost []  Id:     4
  # Query_time: 0.871388  Lock_time: 0.000165 Rows_sent: 46548  Rows_examined: 4000000
  SET timestamp=1653055673;
  select *  from student where stuno >3453451 and stuno < 3500000

Count: 1  Time=0.00s (0s)  Lock=0.00s (0s)  Rows=0.0 (0), 0users@0hosts
  # User@Host: skip-grants user[root] @ localhost []  Id:     4
  # Query_time: 1.807568  Lock_time: 0.000117 Rows_sent: 4000000  Rows_examined: 4000000
  SET timestamp=1653055064;
  select * from student

Count: 1  Time=0.00s (0s)  Lock=0.00s (0s)  Rows=0.0 (0), 0users@0hosts
  # User@Host: skip-grants user[root] @ localhost []  Id:     4
  # Query_time: 171.162157  Lock_time: 0.000060 Rows_sent: 0  Rows_examined: 0
  use school;
  SET timestamp=1653055001;
  CALL insert_stu1(100001,4000000)
```

#### 关闭慢查询日志参数

```mysql
mysql>  set global slow_query_log = "off";
Query OK, 0 rows affected (0.01 sec)

mysql> show variables like "%slow_query_log%";
+---------------------+-----------------------------------+
| Variable_name       | Value                             |
+---------------------+-----------------------------------+
| slow_query_log      | OFF                               |
| slow_query_log_file | /home/mysql/data/testttt-slow.log |
+---------------------+-----------------------------------+
2 rows in set (0.00 sec)
```

#### 删除慢查询日志参数

### 5.查询SQL执行成本：SHOW PROFILE

SHOW PROFILE当前已废弃

### 6.[分析查询语句：EXPLAIN](https://segmentfault.com/a/1190000022696458)

<font color="red">定位到查询慢的SQL之后，我们就可以使用explain工具做针对性的分析查询语句</font>

explain能做是什么：

1. 表的读取顺序
2. 数据读取操作的 操作类型
3. 哪些索引可以使用
4. <font color="red">哪些索引被实际使用</font>
5. 表之间的引用
6. <font color="red">每张表有多少行被优化器查询</font>

**explain各列的作用**

```mysql
mysql> explain select *  from student where stuno >3453451 and stuno < 3500000;
+----+-------------+---------+------------+------+---------------+------+---------+------+---------+----------+-------------+
| id | select_type | table   | partitions | type | possible_keys | key  | key_len | ref  | rows    | filtered | Extra       |
+----+-------------+---------+------------+------+---------------+------+---------+------+---------+----------+-------------+
|  1 | SIMPLE      | student | NULL       | ALL  | NULL          | NULL | NULL    | NULL | 3989881 |    11.11 | Using where |
+----+-------------+---------+------------+------+---------------+------+---------+------+---------+----------+-------------+
1 row in set, 1 warning (0.00 sec)
```

| 列名          | 描述                                                   |
| ------------- | ------------------------------------------------------ |
| id            | 在一个大的查询语句总每个select都对应一个select id      |
| select_type   | 查询类型                                               |
| table         | 表明                                                   |
| partitions    | 匹配的区分信息                                         |
| type          | 针对单表的访问方法                                     |
| possible_keys | 可能用到的索引                                         |
| key           | 实际使用到的索引                                       |
| key_len       | 实际使用到的索引的长度                                 |
| ref           | 当使用索引列等值查询时，与索引列进行等值匹配的对象信息 |
| rows          | 预估的需要读取的记录条数                               |
| filtered      | 某个表经过搜索条件过滤之后剩余记录条数的百分比         |
| Extra         | 一些额外的信息                                         |

1. table

2. id

   - id如果相同，可以认为是一组，从上往下顺序执行
   - 在所有组中，id值越大，优先级越高，越先执行
   - 关注点：id号每个号码，表示一趟独立的查询, 一个sql的查询趟数越少越好

3. select_type

4. partitions

5. <font color="red">type</font>

   执行计划的一条记录就代表着MySQL对某个表的的访问类型，其中type表示的是具体的访问类型，是一个重要的指标。比如看到type列的值是ref，表明mysql即将使用ref的方式对表进行查询。

   完整的访问方法如下：`system`、`const`、`eq_ref`、`ref`、`fulltext`、`ref_or_null`、`index_merge`、`unique_subquery`、`index_subquery`、`range`、`index`、`all`。访问方法越靠前表明执行效率越高

   - system

   - const

     当我们根据主键或者唯一二级索引与常数进行等值匹配是，对表的访问方式是`const`

   - eq_ref

     在进行连接查询时，如果被驱动表是通过主键或者唯一二级索引进行等值匹配时，对该驱动表的访问方式是`eq_ref`

   - ref

     当我们通过普通二级索引与常量进行等值匹配来查询某个表示，那么对该表的访问方式就可能是`ref`

   - index_merge

   结果值从最好到最坏依次是：

   <font color="red">`system`、`const`、`eq_ref`、`ref`</font>、`fulltext`、`ref_or_null`、`index_merge`、`unique_subquery`、`index_subquery`、<font color="red">`range`、`index`、`all`</font>

   其中最重要的提取出来（红色标注）。SQL性能优化的目标：至少要得到range级别，要求是ref级别，最好是const级别。

6. possible_keys和key

   possible_keys表示的是在某个查询中可能会用到的索引，key列表示的是查询中实际用到的索引，如果为null则表示没有使用到索引。

   ![](https://i.bmp.ovh/imgs/2022/05/21/0cfd3dab46034079.png)

7. <font color="red">key_len</font>

   key_len表示的是实际使用到的索引的长度（单位：字节），可以帮我们检查是否充分使用到了索引。主要是针对联合索引有一定的参考意义，值越大越好。

8. ref

   当使用索引列进行等值查询时，与索引列进行等值匹配的对象的信息。

9. <font color="red">rows</font>

   预估的需要读取的记录条数，值越小越好（越小的话对应的记录的数据更有可能在同一个页中，此时IO的次数就会相对更少）

10. filtered

11. <font color="red">extra</font>

    <font color="red">一些额外的信息，我们可以通过外的信息来更加准确的理解MySQL到底是如何执行给定的查询语句的。</font>

12. 小结

### 7.EXPLAIN的进一步使用

这里谈谈EXPLAIN的输出格式。EXPLAIN可以输出四种格式： 传统格式 ， JSON格式 ， TREE格式 以及 可 视化输出 。用户可以根据需要选择适用于自己的格式。

1. 传统格式

2. JSON格式

   JSON格式：在EXPLAIN单词和真正的查询语句中间加上 FORMAT=JSON 。

   ```mysql
   EXPLAIN FORMAT=JSON SELECT ....
   ```

3. TREE格式

4. 可视化输出格式

### 8.分析优化器的执行计划：trace

### 9.MySQL监控分析视图-sysschema

## 第10章_索引优化和查询优化

数据库调优的维度：

- 索引失效，没有充分利用到索引    -> 索引建立
- 关联太多join（设计缺陷或者不得已的需求） ->   SQL优化
- 服务器调优以及参数配置（缓冲、线程数）   -->  调整my.cnf配置文件
- 数据过多  ->  分库分表

SQL优化的两个方向：

- 物理查询优化

  物理查询优化是通过<font color="red">索引</font>和<font color="red">表连接</font>的等技术进行优化，这里需要重点掌握索引的使用

- 逻辑查询优化

  逻辑查询优化是通过SQL等价变化提高查询效率（换一种执行效率更高的SQL）

1. 

### 1.数据准备

### 2.索引失效案例

#### 最佳左前缀法则

MySQL可以为多个字段创建索引，一个索引最多可以包含16个字段。<font color="red">对于联合索引，过滤条件如果要正确使用索引，必须按照索引建立时的顺序使用。一旦跳过某个字段，索引后面的字段都将无法使用。</font>

如果查询条件没有联合索引的第一个字段时，那么建立的联合索引将全部失效

> 拓展：
>
> 索引文件具有B+树的最左匹配特性，如果左边的值未确定，那么无法使用该索引。

#### 主键插入顺序

我们自定义的主键列id拥有AUTO_INCREMENT属性，在插入记录时存储引擎会自动为我们填入自增的主键值。这样的<font color="red">主键占用空间小，可以顺序写入，减少页分裂。</font>

#### 计算、函数会让索引失效

```mysql
# 可以正常使用索引
select * from student where student.name like "abc%";
# 因为使用了left函数，底层进行了全表扫描，最终索引失效
select * from student where left(student.name, 3) = "abc";
```

```mysql
# 因为在索引字段使用了运算，所以索引失效
EXPLAIN SELECT SQL_NO_CACHE id, stuno, NAME FROM student WHERE stuno+1 = 900001;
```

![image-20220522160739185](C:\Users\lucas.zhao\AppData\Roaming\Typora\typora-user-images\image-20220522160739185.png)

#### 类型转换（自动转换或者手动转换）会让索引失效

```mysql
# 索引失效（因为name为字符串类型，这里传入了整形，存在隐式类型转换问题，最终导致索引失效）
EXPLAIN SELECT SQL_NO_CACHE * FROM student WHERE name=123;
```

![](https://i.bmp.ovh/imgs/2022/05/22/524a37756753bda5.png)

#### 范围查询条件右边的列索引失效

```mysql
# 针对age、name、classid字段建立联合索引
create index idx_age_name_classid on student(age,classid,name);
```

```mysql
EXPLAIN SELECT SQL_NO_CACHE * FROM student WHERE student.age=30 AND student.classId>20 AND student.name = 'abc' ;
```

![](https://i.bmp.ovh/imgs/2022/05/22/e2a83422e3f1cdee.png)

执行计划显示key_len为10，联合索引中实际上就使用到了age、classid索引，<font color="red">因为classid字段使用了范围查询，其后的字段索引会失效。</font>

解决方式：

- 在创建联合索引时，可以把等值查询的字段放在锁边，范围查询的字段放在最后，这样创建的联合索引最终都是有效的。
- 将范围查询放在where条件的最后

#### 不等于（!= 或者 <>）索引失效

#### is null可以使用索引，is not null无法使用索引

最好<font color="red">在设计数据库表时将字段设置成not null约束</font>。或者给字段设置默认值：int类型默认值为0，字符串类型的默认值为空字符串。

<font color="red">同理，not like也无法使用索引，最终将会导致全表扫描。</font>

#### like通配符以%开头的索引失效

> 拓展：alibaba《Java开发手册》
>
> 【强制】页面搜索严禁左模糊或者全模糊，如果需要请走搜索引擎来解决。

#### OR前后存在非索引列，索引失效

```mysql
EXPLAIN SELECT SQL_NO_CACHE * FROM student WHERE age = 10 OR classid = 100;
```

age字段有索引（走索引）、classid字段没有索引（全表扫描），通过or的方式查询，最终还是全表扫描。

#### 数据库和表的字符集统一使用uft8mb4

#### 练习及一般性建议

假设建立联合索引 index(a, b, c)

![](https://i.bmp.ovh/imgs/2022/05/22/4ee613ab9c5265c5.png)

### 3.关联查询优化

#### 数据准备

#### 采用左外连接

```mysql
EXPLAIN SELECT SQL_NO_CACHE * FROM `type` LEFT JOIN book ON type.card = book.card;
```

![Snipaste_2022-05-28_12-07-08](https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/Snipaste_2022-05-28_12-07-08.3js22ouilpg0.webp)

结论：没有添加索引，type类型为All，走的是全表查询

```mysql
# 给被驱动表添加索引(可以避免全表扫描)
ALTER TABLE book ADD INDEX Y ( card); 
EXPLAIN SELECT SQL_NO_CACHE * FROM `type` LEFT JOIN book ON type.card = book.card;
```

可以看到第二行的 type 变为了 ref，rows 也变成了优化比较明显。这是由左连接特性决定的。<font color="red">LEFT JOIN条件用于确定如何从右表中进行搜索，左表的数据一定都有，所以右表即被驱动表使我们优化的关键，一定要添加索引。</font>

#### 采用内连接

1. 对于内连接查询，查询优化器可以决定谁作为驱动表，谁作为被驱动表
2. 对于内连接查询，如果表的连接条件中只有一个字段有索引，那么有索引的字段所在的表会被作为被驱动表
3. 对于内连接查询，如果表的连接条件都存在索引的情况下，优化器会选择小表作为驱动表。<font color="red">即小表驱动大表</font>

#### join原理

### 4.子查询优化

### 5.排序优化

### 6.GROUP BY优化

### 7.优化分页查询

### 8.优先考虑覆盖索引

### 9.如何给字符串添加索引

### 10.索引下推

### 11.普通索引vs唯一索引

### 12.其他查询优化策略

### 13.淘宝数据库，主键如何设计











<font color="red"></font>