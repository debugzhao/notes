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

##### 1.一个简单的索引设计方案

为了快速定位数据，我们需要建立一个目录，建立这个目录必须符合以下特点：

1. 下一个数据页中的数据的主键必须大于上一个数据页中的数据主键
2. 给所有也简历一个目录项

![image](https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/image.43n3prsata20.webp)

这个目录还有一个别名就是**索引**。

##### 2.InnoDB中的索引方案

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

##### 1.聚簇索引

##### 2.二级索引

> 二级索引也称为辅助索引、非聚簇索引

##### 3.联合索引

#### 3.4 InnoDB的B+树索引的注意事项

### 4.MyISAM的索引方案

#### 4.1 MyISAM索引的原理

#### 4.2 MyISAM和InnoDB对比

### 5.索引的代价

### 6.MySQL数据结构选择的合理性

#### 6.1 全表遍历

#### 6.2 Hash结构

#### 6.3 二叉搜索树

#### 6.4 AVL树

#### 6.5 B-Tree

#### 6.6 B+Tree

#### 6.7 R树













<font color="red"></font>