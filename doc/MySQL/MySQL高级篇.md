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

<font color="red">Join方式连接多个表，本质上就是多个表的数据之间的循环匹配。</font>

**Index Nested_Loop Join（索引嵌套循环连接）**

INLJ其优化的思路就是为了减少内层表数据的匹配次数，所以要求被驱动表必须有索引才行。通过外层表匹配条件直接与内层表索引进行匹配，避免直接和内层表的每条记录进行匹配。

**Join小结**

1. 整体查询效率：INLJ > BNLJ > SNLJ
2. 永远用小结果集驱动大结果集（本质上是减少外层循环数据量）
3. 为被驱动表的连接条件添加索引
4. 减少驱动表不必要的字段查询
5. 需要JOIN的字段，数据类型保持绝对一致
6. LEFT JOIN时，选择小表作为驱动表，大表作为被驱动表。减少外层循环的次数
7. INNER JOIN时，MySQL会选择将小结果集作为驱动表。
8. 能够直接多表关联的查询尽量选择关联查询，减少子查询

### 4.子查询优化

子查询即一个SELECT的查询结果可以作为另外一个SELECT语句的查询条件，子查询可以一次性完成逻辑上需要多个操作才能完成的SQL操作。

虽然子查询可以帮助我们完成复杂的查询，但是子查询的查询效率并不高，原因如下：

1. <font color="red">子查询会建立临时表</font>

   执行子查询时，MySQL会为内层的查询结果建立一个临时表，然后外层查询从零时表中查询记录。查询完毕时，再撤销这些临时表。这样会消耗过多的CPU资源和IO资源，导致大量的慢查询

2. <font color="red">子查询的结果集不存在索引</font>

   子查询的结果存在临时表中，无论是内存临时表还是磁盘临时表都不会存在索引，所以查询效率会受到一定影响

3. <font color="red">子查询性能受结果集大小影响</font>

   对于返回结果集比较大的子查询，其查询性能影响也会比较大

<font color="red">不建议使用子查询，建议将子查询SQL拆开结合程序进行多次查询，或者使用连接查询代替</font>

> 小提示：尽量不要使用NOT IN或者NOT EXISTS，用LEFT JOIN xxx ON xxx WHERE xx IS NULL代替

### 5.排序优化

Q：在WHERE条件字段上添加索引之后，为什么还要在ORDER BY字段加索引呢？

A：在MySQL中，默认支持两种排序方式，分别是`FileSore`和`Index`排序

- Index排序：索引可以保证数据的有效性，不需要再进行排序，效率更高
- FileSort排序：FileSort排序一般在内存中进行排序，占用CPU较多。如果待排序结果较大，会产生临时文件IO到磁盘中进行排序的情况，效率较低

**优化建议：**

1. 在进行SQL查询时，可以在WHERE和ORDER BY子句中使用索引，目的是<font color="red">在WHERE子句中避免进行全表扫描，在ORDER BY子句中避免使用FileSort排序</font>
2. 尽量使用Index完成ORDER BY排序。如果WHERE和ORDER BY是相同的列则使用单列索引，如果不相同则可以考虑使用联合索引

#### order by时不limit，索引失效

```mysql
# 创建索引
CREATE INDEX index_age_classid_name on student(age, classid, name);
# 没有limit限制，索引失效
# order by时走的是(age, classid)二级索引，但是由于查询的是全部字段，
# 还需要进行回表操作，性能还不如全表扫描，最终优化器不适用索引，进行全表扫描
EXPLAIN SELECT SQL_NO_CACHE * FROM student ORDER BY age, classid;
# 进行了limit限制，只需要对前10条数据进行回表即可，最终走的是联合索引
EXPLAIN SELECT SQL_NO_CACHE * FROM student ORDER BY age, classid LIMIT 10;
```

#### order by时顺序错误（不符合最左匹配原则），索引失效

```mysql
# 创建索引
CREATE INDEX index_age_classid_name on student(age, classid, name);
# 索引有效
EXPLAIN SELECT * FROM student ORDER BY classid LIMIT 10;
# 索引失效(不符合联合索引的最左匹配原则)
EXPLAIN SELECT * FROM student ORDER BY classid, name LIMIT 10;
```

#### order by时排序顺序不一致，索引失效

```mysql
# 创建索引
CREATE INDEX index_age_classid_name on student(age, classid, name);
CREATE INDEX index_age_classid_stuno on student(age, classid, stuno);
# 一个升序、一个降序，索引失效
EXPLAIN SELECT * FROM student ORDER BY age DESC classid ASC LIMIT 10;
# 不符合最左匹配原则，索引失效
EXPLAIN SELECT * FROM student ORDER BY classid DESC name DESC LIMIT 10;
# 两个降序，索引有效
EXPLAIN SELECT * FROM student ORDER BY age DESC classid DESC LIMIT 10;
```

#### 无过滤，不索引

### 6.GROUP BY优化

- GROUP BY使用索引的原则几乎和ORDER BY一致，可以直接使用索引
- GROUP BY先排序再分组，遵照索引建的最左匹配原则
- <font color="red">WHERE使用效率高于HAVING，能使用WHERE就不使用HAVING</font>
- ORDER BY、GROUP BY、DISTINCT这些语句比较消耗CPU，可以选择让代码去实现类似的功能

### 7.优化分页查询

一般的分页查询，通过创建覆盖索引可以很好的提高性能。但是还是不能避免一个非常常见又头疼的问题比如：`limit 2000000, 10`，此时MySQL需要对2000010条记录排序，仅仅返回第2000000 ~ 2000010条记录，其他的则丢弃掉，排序查询的代价是非常大的

**优化方式：**

1. 先在索引上完成排序分页操作，最后根据主键关联会原表查询原表所需要的内容

   ```mysql
   EXPLAIN SELECT * FROM student t, (SELECT id FROM student ORDER BY id LIMIT 2000000, 10) id_temp_table WHERE t.id = id_temp_table.id
   ```

   ![image](https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/image.38ab87ckhp80.webp)

2. 该方案适合主键自增的表，可以把LIMIT查询转换成某个范围查询

   ```mysql
   EXPLAIN SELECT * FROM student WHERE id > 2000000 LIMIT 10;
   ```

### 8.优先考虑覆盖索引

#### 什么是覆盖索引？

一个索引包含了满足查询结果的所有数据就叫做覆盖索引

#### 覆盖索引的利弊

**优点：**

1. 覆盖索引可以避免回表查询操作
2. 可以把随机IO变成顺序IO，加快查询效率

### 9.如何给字符串添加索引

### 10.索引下推

### 11.普通索引vs唯一索引

### 12.其他查询优化策略

#### EXISTS和IN的区分

##### EXISTS介绍

EXISTS对外表用loop逐条查询，每次查询都会查看exists的条件语句，当exists的条件语句能够返回记录行时（无论行数是多少，只要能返回）条件就为真，返回当前loop到的这条记录；反之exists里的条件为假即不能返回记录行，则外表loop到的这条记录就被丢弃。exists的条件就像一个bool条件，当能返回一个结果集时就为true，不能返回结果集时就为false

```mysql
select * from user where exists (select 1);
```

对user表的记录逐条取出，由于子条件中的`select 1`永远能返回记录行，那么user表的所有记录都将被加入结果集，所以与`select * from user;`是一样的。

**结论：如果A表中有n条记录，那么exists查询就是将这n条记录逐条取出，然后判断n遍exists条件**

##### IN介绍

in查询就是相当于多个or条件查询的叠加

```mysql
select * from user where user_id in (1, 2, 3);
# 等效于
select * from user where user_id = 1 or user_id = 2 or user_id = 3;
```

**结论：in查询就是将子查询的条件的记录全部查询出来，假设结果集为B，共有m条记录，然后再将查询条件的结果集分解成m个，再进行m次查询。**

##### 使用上的区别

in查询的子条件返回结果必须只有一个字段，例如

```mysql
select * from user where user_id in (select id from B);
```

不能是多个字段，例如：

```mysql
select * from user where user_id in (select id, age from B);
```

但是exists就没有查询字段个数的限制

##### 结论

<font color="red">MySQL中in语句是把外表和内表做join连接；而exists是对外表做loop循环，每次循环再对exists的条件进行判断。</font>

1. <font color="red">如果查询的两个表大小相当，那么用in和exists差别不大</font>
2. <font color="red">如果两个表中一个表大一个表小，那么IN适合外表大而子表小的情况；EXISTS适合外表小而子表大的情况</font>

#### COUNT(*)和COUNT(具体字段)效率

在 MySQL 中统计数据表的行数，可以使用三种方式： SELECT COUNT(*) 、 SELECT COUNT(1) 和 SELECT COUNT(具体字段) ，使用这三者之间的查询效率是怎样的？

#### 关于SELECT(*)

在表查询中，建议明确查询字段，不要使用*作为查询的字段列表，原因如下：

1. MySQL在解析过程中，会通过`查询数据字典`将 * 转换成所有列，转换过程中会大大消耗资源和时间
2. 无法使用`聚簇索引`

#### LIMIT 1对优化的影响

LIMIT 1对优化的影响针对的是会扫描全表的SQL语句，如果可以确定查询结果只有一条，那么加上`LIMIT 1`时，当找到结果的时候就不会继续扫描了，可以加快查询时间

如果数据库表已经对字段建立了唯一索引，那么可以通过索引进行查询，不需要加上`LIMIT 1`

#### 多使用COMMIT

### 13.淘宝数据库，主键如何设计

#### 主键自增ID的问题

自增ID做主键，简单易懂，几乎所有数据库都支持自增类型，只是实现上各自有所不同而已。自增ID除 了简单，其他都是缺点，总体来看存在以下几方面的问题

1. 可靠性不高

   存在自增ID回溯的问题

2. 安全性不高

   对外暴露的接口可以非常容易猜测对应的信息。比如：/User/1/这样的接口，可以非常容易猜测用户ID的 值为多少，总用户数量有多少，也可以非常容易地通过接口进行数据的爬取。

3. 性能差

   自增ID的性能较差，需要在数据库服务器端生成。

4. 交互多

5. 局部唯一性 

   最重要的一点，自增ID是局部唯一，只在当前数据库实例中唯一，而不是全局唯一，在任意服务器间都 是唯一的。对于目前分布式系统来说，这简直就是噩梦。

#### 业务字段做主键

<font color="red">建议尽量不要用跟业务有关的字段做主键。毕竟，作为项目设计的技术人员，我们谁也无法预测 在项目的整个生命周期中，哪个业务字段会因为项目的业务需求而有重复，或者重用之类的情况出现</font>

#### 淘宝的主键设计

从上图可以发现，订单号不是自增ID！我们详细看下上述4个订单号：

```java
1550672064762308113
1481195847180308113
1431156171142308113
1431146631521308113
```

大胆猜测，淘宝的订单ID设计应该是：

```java
订单ID = 时间 + 去重字段 + 用户ID后6位尾号
```

<font color="red">这样的设计能做到全局唯一，且对分布式系统查询及其友好。</font>

#### 推荐的主键设计

1. 核心业务

   对应表的主键自增ID，如告警、日志、监控等信息。

2. 非核心业务

   主键设计至少应该是全局唯一且是单调递增。全局唯一保证在各系统之间都是唯一的，单调 递增是希望插入时不影响数据库性能。

## 第12章数据库其他调优策略

### 3.优化数据库结构

#### 拆分表：冷热数据分离

拆分表的思路：把1个包含很多字段的表拆分2个或者多个相对比较小的表，进行热数据（经常要查询和更新的数据）和冷数据（使用频率比较低的字段）的隔离，减小表的宽度。如果很多字段都放在一个表中，每次查询都要取很多记录，可能会消耗较多的资源。在mysql中如果一个表越宽，把表装进内存缓冲池时所占用的内存也就越大，也会消耗更多的IO。

冷热数据分离的目的：

1. 减少磁盘IO，可以保证热数据的内存缓存命中率
2. 更有效地利用缓存，避免读入无用的冷数据

1. 

#### 增加中间表

对于经常要联合查询的表，可以建立中间表以提高查询效率

#### 增加冗余字段

适当增加冗余字段以减少联合查询的操作，提高查询效率

#### 优化数据类型

优先选择适合存储需要的占用空间最小的数据类型

1. 避免使用TEXT、BLOB等数据类型
2. 避免使用ENUM枚举类型，可以考虑使用TINYINT代替枚举类型
3. 使用TIMESTAMP存储时间。TIMESTAMP占用4字节、DATETIME占用8字节空间。另外TIMESTAMP具有自动更新以及自动赋值的特性。
4. 使用DECIMAL代替FLOATfa和DOUBLE存储精确浮点数

#### 优化插入记录的速度

#### [使用非空约束](https://segmentfault.com/a/1190000039774659)

在设计字段时，如果业务允许，建议尽量使用非空约束

1. 在进行比较和计算时，省去对NULL值的字段判断是否为空的开销，提高存储效率（可为NULL的列使得索引、索引统计和值比较都更复杂）
2. 非空字段容易创建索引，索引NULL列需要额外的空间来保存

### 4.大表优化

#### 限定查询的范围

#### 读写分离

主库负责写，从库负责读取数据

1. 一主一从模式

   ![image](https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/image.32qmkg73wkk0.webp)

2. 双主双从模式

   ![image](https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/image.1aub6ttdkr1c.webp)

#### 垂直拆分

当数据量级达到 千万级 以上时，有时候我们需要把一个数据库切成多份，放到不同的数据库服务器上， 减少对单一数据库服务器的访问压力

![image](https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/image.1wl74rka9qsg.webp)

**垂直拆分优点：**

可以使得列数据变小，在查询时减少读取的block数，减少IO次数

**垂直拆分缺点：**

垂直拆分会出现主键冗余的情况，需要额外管理冗余的主键。垂直拆分会让事务变得更加复杂

#### 水平拆分

![image](https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/image.11a6g5gw28kw.webp)

下面补充一下数据库分片的两种常见方案

1. 客户端代理

    分片逻辑在应用端，封装在jar包中，通过修改或者封装JDBC层来实现。

2. 中间件代理

   在应用和数据中间加了一个代理层。分片逻辑统一维护在中间件服务中

### 5.其他调优策略

#### 服务器语句超时处理

## 第13章 事务基础知识

### 1.数据库事务概述

#### 存储引擎支持情况

在MySQL中只有InnoDB存储引擎是支持事务的。

#### 基本概念

事务概念：事务是<font color="red">一组逻辑操作单元</font>，使数据从一种状态变换到另一种状态。

事务处理的原则：保证所有事务都作为<font color="red">一个工作单元</font>来执行，即使出现了故障，也不能改变这种执行方式。要么所有的事务<font color="red">全被提交（commit）</font>，要么整个事务<font color="red">全被回滚（rollback）</font>到最初的状态。

```mysql
# 案例：用户A向用户B转账100元。这两个操作被看做一个事务，要不全部执行，要么都不执行
update account set money = money - 100 where name = 'A';
update account set money = money + 100 where name = 'B';
```

#### 事务的ACID特性❗️❗️

- 原子性（atomicity）

  原子性是指事务是一个不可分割的工作单位，要么全部提交，要么全部回滚失败，不会存在中间状态

- 一致性（consistency）

  一致性指的是事务执行前后，数据从一个`合法性状态`转变成另外一个`合法性状态`。这里的合法性指的是语义上的合法性而不是语法上的合法性，和具体的业务有关。

  <font color="red">满足预定义的约束的数据就叫做合法状态的数据。</font>

- 隔离性（isolation）

  <font color="red">事务的隔离性指的是一个事务的执行不能被其他事务干扰，即一个事务内部操作的数据对被其他的并发的事务是隔离的，并发执行的各个事务不能相互干扰。</font>

  如果无法保证隔离性会怎么样？假设A账户有200元，B账户0元。A账户往B账户转账两次，每次金额为50 元，分别在两个事务中执行。如果无法保证隔离性，会出现下面的情形：

  ```mysql
  UPDATE accounts SET money = money - 50 WHERE NAME = 'AA';
  UPDATE accounts SET money = money + 50 WHERE NAME = 'BB'
  ```

  <font color="red">如上图所示，无法保证事务的隔离性，事务2先执行完将B的50的值写回磁盘，这时轮到事务1执行，将磁盘中的50更改为50。这时就会出现问题。</font>

- 持久性（durability）

  持久性是指一个事务一旦提交，他对数据库中的数据的修改是<font color="red">永久的</font>，即便数据库出现故障也不会对其产生影响。

  持久性是通过<font color="red">事务日志</font>来保证的。日志包括<font color="red">重做日志</font>和<font color="red">回滚日志</font>。当数据库崩溃或者服务器宕机时，进程重启会先扫描日志恢复数据，从而使事务具有持久性。

#### 事务的状态

### 2.如何使用事务

#### 显式事务

#### 隐式事务

#### 隐式提交数据的情况

#### 使用举例1：提交与回滚

#### 使用举例2：测试不支持事务的engine

#### 使用举例3：SAVEPOINT

### 3.事务隔离级别

#### 数据准备

#### 数据的并发问题❗️❗️

针对事务的隔离性和并发性，我们怎么做取舍呢？先看一下访问相同的数据的事务在<font color="red">**并行执行情况下可能出现的问题**</font>

1. 脏写

   对于两个事务SessionA、SessionB，如果SessionA<font color="red">修改了</font>另一个<font color="red">未提交</font>的事务SessionB<font color="red">修改过</font>的数据，那就意味着发生了<font color="red">脏写</font>

   ![](https://i.bmp.ovh/imgs/2022/06/14/c9179ce09301056f.png)

2. 脏读

   对于两个事务SessionA、SessionB，SessionA读取了Session B已经更新但是还没有提交的数据，之后如果Session B回滚，那么Session A读取的数据就是临时且无效的。

3. 不可重复读

   对于两个事务SessionA、SessionB，Session A读取了一个字段，然后Session B更新了该字段，之后Session A再次读取同一个字段，值就不同了。这就意味发生了不可重复读。

4. 幻读

   对于两个事务SessionA、SessionB，Session A从一个表中读取了一个字段，然后Session B在该表中插入了一些新的行。之后Session A如果再次读取同一个表，就会多出来几行数据。这就意味着发生了幻读。

#### 数据库系统的四个隔离级别❗️❗️

上面介绍了四种事务在并发过程中遇到的问题，这四种问题有轻重缓急之分，按照严重性排序依次是：

<font color="red">**脏写 > 脏读 > 不可重复读 > 幻读**</font>

我们愿意舍弃一些隔离性来换取一些性能就在这里体现：设置一些隔离级别，隔离级别越低，出现的并发问题就越多。<font color="red">SQL表中设立了四个隔离级别。</font>

1. 读未提交（READ UNCOMMITTED）

   在该隔离级别中， 所有事务都可以看到其他未提交事务的执行结果。不能避免脏读、不可重复度、幻读。

2. 读已提交（READ COMMITTED）

   读已提交满足了隔离级别的简单定义：一个事务只能看见其他已提交事务做的改变。这是大多数数据库系统的隔离级别（但是不是MySQL默认的隔离级别）。可以避免脏读，但是不能避免不可重复度、幻读。

3. 可重复读（REPEATABLE READ）

   事务A读取到了一条数据之后，事务B对其进行了修改并提交，此时事务A再次读取该数据，读到的还是原来的内容。可以避免脏读、不可重复读，但是幻读的现象依然存在。（<font color="red">可重复读是MySQL默认的隔离级别</font>）

4. 可串行化（SERIALIZABLE）

   可串行化，确保事务可以从一个表中读取相同的行。在这个事务执行期间，禁止其他事务对该表进行修改、插入、删除操作。所有的并发问题都可以避免，但是性能十分低下。

不同的隔离级别有不同的现象，并有不同的锁和并发机制，隔离级别越高，数据库的并发性能就越差。

#### MySQL支持的四种隔离级别

MySQL的默认隔离级别为`REPEATABLE READ`

```mysql
mysql>  SHOW VARIABLES LIKE 'transaction_isolation';
+-----------------------+-----------------+
| Variable_name         | Value           |
+-----------------------+-----------------+
| transaction_isolation | REPEATABLE-READ |
+-----------------------+-----------------+
1 row in set (0.08 sec)
```

#### 如何设置事务的隔离级别

```mysql
SET [GLOBAL|SESSION] TRANSACTION_ISOLATION = '隔离级别'
#其中，隔离级别格式：
> READ-UNCOMMITTED
> READ-COMMITTED
> REPEATABLE-READ
> SERIALIZABLE
```

> 小结： 数据库规定了多种事务隔离级别，不同隔离级别对应不同的干扰程度，隔离级别越高，数据一致性就越好，但并发性越弱。

#### 不同隔离级别举例

### 4.事务的常见分类

- 扁平事务
- 带有保存点的扁平事务
- 分布式事务

## 第14章 MySQL事务日志

事务有4中特性：原子性、一致性、隔离性和持久性。那么事务的四种特性是由什么机制保证的呢？

1. 事务的隔离性有锁机制实现
2. 事务的原子性、一致性、持久性由事务的redo log和undo log保证的
   1. redo log称为重做日志，提供写入再操作，恢复提交事务修改的页操作，用来保证事务的持久性
   2. undo log称为回滚日志，回滚行记录到某个特定的版本，用来保证事务的原子性、一致性。

### 1.redo log

#### 为什么需要redo log

MySQL每次修改数据的时候会先将数据读取到内存中（缓冲池中），可以消除CPU和磁盘之间读写速度之间的鸿沟。但是数据写入到内存会带来掉电即失的问题。于是MySQL引入`checkpoint机制`，即每隔一段时间将内存中的数据写入到日志中（redo log）。checkpoint机制可以保证最终数据的落盘，但是checkpoint不是每次变更就触发的（每次变更触发会非常消耗性能，并且也没有必要），所以还是存在数据丢失的问题。

所以redo log的作用是保证事务的持久性

![](https://i.bmp.ovh/imgs/2022/06/16/cce50a1e071138bb.png)

#### redo log好处、特点

好处

1. redo日志降低了刷盘频率，每隔一段时间才会触发checkpoint
2. redo日志占用的空间非常小

特点

1. redo日志是顺序写入磁盘的，比随机写效率高
2. 事务执行的过程中，redo log不断记录

#### redo log的组成

1. redo log buffer 保存在内存中，是易失的。
2. redo log file 保存在硬盘中，是持久的。

#### redo的整体流程

![](https://i.bmp.ovh/imgs/2022/06/16/c1f560b83b640b99.png)

1. 先将原始数据从磁盘读取到内存中，修改数据的内存拷贝
2. 生成一条重做日志并写入到redo log buffer中，保存的是数据修改之后的值
3. 当事务commit时，将redo log buffer中的内容刷新到redo log file中（redo log file写入的方式是追加写的）
4. 定期将内存中修改的数据刷新到磁盘中

#### redo log的刷盘策略

#### 不同刷盘策略演示

#### 写入redo log buffer过程

#### redo log file

### 2.undo log

redo log是事务持久新的保证， undo log是事务原子性的保证。<font color="red">事务更新数据的前置操作需要先写入一个undo log。</font>

#### 如何理解undo log

事务需要保证 原子性 ，也就是事务中的操作要么全部完成，要么什么也不做。但有时候事务执行到一半 会出现一些情况，比如：

1. 事务在执行过程中可能会遇到各种错误：服务器本身错误、操作系统异常、甚至是断电异常
2. 开发者在事务的执行过程中手动输入`ROLLBACK` 提前结束当前事务的执行

出现以上情况时我们就需要把数据恢复成原来的数据，我们称之为回滚。这样就可以造成一个假象，这个事务看起来什么都没有做，所以符合`原子性`要求。

#### undo 日志的作用

1. 回滚数据
2. MVCC

#### undo的存储结构

#### undo的类型

1. insert undo log

#### undo log的生命周期

##### 简要生成过程

##### 详细生成过程

##### undo log是如何回滚的

##### undo log的删除

## 第15章 锁

MySQL事务的`隔离性`是由`锁机制`来实现的

### 1.概述

<font color="red">锁是计算机为了保证数据的一致性和安全性，协调多个线程或者进程并发访问某一资源的机制。</font>锁冲突也是影响数据库并发访问性能的一个重要因素。

### 2.MySQL并发事务访问相同记录的三种情况

#### 2.1 读-读情况

并发事务同时读取相同数据的时候，不会出现数据不一致问题，所以允许这种情况发生。

#### 2.2 写-写情况

写写情况，即并发事务同时对相同数据做出改动。这种情况下会发生<font color="red">脏写</font>，任何一种隔离级别都不允许脏写问题的发生。所以在多个未提交的事务同时对同一条记录做出修改时，需要让它们<font color="red"></font>排队执行，这里的排队的过程本质上是通过锁机制来实现的。（这里的锁其实是内存的一个数据结构）。

当一个事务对一条记录做写操作时，会先到内存中检查与该记录关联的锁结构，如果没有与之关联的锁结构则生成一个锁结构与之关联。如果有关联的锁结构，则检查该锁结构的等待状态，不能进行写操作则进入等待状态。

![](https://i.bmp.ovh/imgs/2022/06/18/928c918d379e3f5a.png)

#### 2.3 读-写情况

读-写或者写-读，即一个事务进行`读操作`、一个事务进行`写操作`，这种情况下就会发生`脏读`、`不可重复读`、`幻读`的问题。

MySQL的`REPEATABLE READ`隔离级别已经解决了`脏读`、`不可重读`的问题。

#### 2.4 并发问题的解决方案

解决脏读、不可重复度、幻读的问题有以下两种解决方案

1. 读操作利用多版本并发控制（MVCC），写操作加锁

   <font color="red">采用MVCC方式，读写操作彼此并不冲突，性能更高，一般情况下我们使用MVCC方式解决并发冲突问题。</font>

2. 读操作、写操作都加锁

   采用加锁方式的话，读写操作都需要排队执行， 影响性能

### 3.锁的不同角度的分类

<img src="https://s3.bmp.ovh/imgs/2022/06/19/19cf061256ee8c2f.png" style="zoom: 80%;" />

#### 3.1 从数据操作的类型划分（读锁、写锁）

- 读锁

  读锁也称为共享锁（Share Lock）。针对同一份数据，多个事务的读操作可以同时进行而不会互相影响，多个事务互相是不阻塞的。

- 写锁

   写锁也称为排它锁（Exclusive Lock）。当前写操作没有完成之前，它会阻止其它的读锁和写锁。这样就能确保在同一时间内只有一个事务在操作，可以防止其他事务读取正在写入的同一资源。

  > 在InnoDB存储引擎中读锁和写锁既可以加在表上也可以加在行上。

#### 3.2 从数据操作的粒度划分

##### 表锁

1. 表级别的S锁、X锁

   都某个表执行SELECT UPDATE  INSERT DELETE语句时，InnoDB存储引擎是不会为这个表提供表锁的。但是在最某个表执行ALTER TABLE或者DROP TABLE这样的DDL语句时，执行了SELECT UPDATE  INSERT DELETE的事务会被阻塞，反之也是如此。这个过程其实是server层使用的元数据（Metadata Lock）锁实现的。

   <font color="red">一般情况下，不会使用InnoDB存储引擎提供的表级别的S锁、X锁，因为性能太低。</font>

2. 意向锁

   InnoDB存储引擎支持同时存在多粒度锁。<font color="red">它允许表锁和行锁共存，</font>而意向锁就是一种<font color="red">表锁</font>。

   意向锁分类

   1. 意向共享锁（Intention Shared Lock, IS）

      事务有意向对表中的某些行添加共享锁

   2. 意向排它锁（Intention Exclusive Lock, IX）

      事务有意向对表中的某些行添加排他锁

   意向锁是存储引擎自己维护的，用户无法手动操作意向锁。意向锁不会与其他的行级的共享锁/排它锁互斥，正是因为如此，意向锁并不会影响多个事务对不同数据行加排它锁的并发性。

   **结论**

   1. InnoDB支持多粒度锁，在某些特定的场景下，行级锁和表级锁可以共存
   2. 意向锁可以在保证并发性的前提下，实现了<font color="red">行锁和表锁共存</font>且<font color="red">满足事务隔离性</font>的要求。

3. 自增锁

4. 元数据锁

   元数据锁即Meta Data Lock，简称MDL锁，属于表锁的范畴。当对一个表做增删改查操作的时候，加 MDL读锁；当要对表做结构变更操作的时候，加 MDL 写 锁。

##### 页级锁

##### 行锁

#### 3.3 从对待锁的态度划分

#### 3.4 按照加锁的方式划分

#### 3.5 其他锁

### 4.锁的内存结构

### 5.锁监控

### 6.附录



<font color="red"></font>



