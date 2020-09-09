### MySQL高级部分

1. mysql内核
2. sql优化
3. mysql服务器优化
4. 各种参数常量设置
5. 查询语句优化
6. 主从复制
7. 软硬件升级
8. 容灾备份
9. sql编程

### MySQL rpm版安装

查看本机是否安装MySQL：

```shell
rpm -qa | grep -i mysql
```

安装MySQL rpm包命令

```shell
rpm -ivh MySQL-client-5.5.48-1.linux2.6.i386.rpm
# -i 自动安装
# -v 显示安装日志
# -h 显示安装进度条哈希值
```

查看MySQL服务时候安装成功

```shell
[root@localhost mysql-5.5-rpm]# mysqladmin --version
mysqladmin  Ver 8.42 Distrib 5.5.48, for Linux on i686
```

启动MySQL服

```shell
service mysql start
```

关闭MySQL服

```shell
service mysql stop
```

给MySQL服务设置用户名和密码

```shell
/usr/bin/mysqladmin -r root password 123456
```

设置MySQL服务开机自动启

```shell
chkconfig mysql on
# 查看MySQL服务在不同方式下的启动方式
chkconfig --list | grep mysql
# 0 1 6为off，表示在不正常方式下不需要自启动，2 3 4 5为正常方式下自启动
mysql          	0:off	1:off	2:on	3:on	4:on	5:on	6:off
```

**使用ntsysv命令以图形化的方式查看系统哪些服务自启动**

### 配置文件

如果需要更改MySQL服务的配置文件，最好先将配置文件进行备份，MySQL5.5的配置文件路径为`/usr/share/mysql/my-huge.cnf`，将其备份到`/etc/my.cnf`

#### 设置服务器字符集编码

进入到`/etc/my.cnf`文件，在相应的节点下进行如下配置

```shell
[client]
default-character-set=utf8

[mysqld]
character_set_server=utf8
character_set_client=utf8
collation-server=utf8_general_ci

[mysql]
default-character-set=utf8
```

#### 相关目录说明

| 路径              | 说明                      |
| ----------------- | ------------------------- |
| /etc/init.d/mysql | 启停相关脚本              |
| /var/lib/mysql    | MySQL数据库文件的存放路径 |
| /usr/share/mysql  | 配置文件目录              |
| /usr/bin          | 命令目录                  |

### 存储引擎

查看MySQL现在支持的存储引擎

```mysql
show engines;
```

查看MySQL当前默认的存储引擎

```mysql
show variables like "%storage_engine%";
```

#### MyISAM和InnoDB

| 对比项   | MyISAM                                           | InnoDB                                                       |
| -------- | ------------------------------------------------ | ------------------------------------------------------------ |
| 主外键   | 不支持                                           | 支持                                                         |
| 事务     | 不支持                                           | 支持                                                         |
| 行表锁   | 即使操作一条记录也会锁住整张表，不适合高并发场景 | 行锁，操作时只锁住某一行，不对其他行产生影响，适合高并发场景 |
| 缓存     | 只缓存索引，不缓存真实数据                       | 不仅缓存索引，还缓存真实数据。对内存要求较高                 |
| 表空间   | 小                                               | 大                                                           |
| 关注点   | 性能                                             | 事务                                                         |
| 默认安装 | 是                                               | 是                                                           |

### SQL性能

#### 常见SQL性能下降原因

- 查询语句写的烂
- 索引失效
- join关联查询太多表
- MySQL服务器调优以及各个参数设置

### Explain执行计划

#### 是什么

使用Explain关键字可以模拟优化器执行SQL查询语句，从而知道MySQL是如何处理器的SQL 查询语句，进而分析出你的查询语句或者表结构的性能瓶颈。

#### 能干嘛

- 表的读取顺序
- 数据读取操作的操作类型
- 哪些索引可以被使用
- 哪些索引被实际使用
- 表之间的引用关系
- 每张表有多少行被优化器查询

#### 如何使用

```mysql
mysql> explain  select * from tbl_dept a left join tbl_emp b on a.id = b.deptId;
```

![](C:\Users\Administrator\Desktop\md\MySQL\images\explain返回结果.png)

##### Explain返回结果关键字分析

- id

  id为一组数字，表示select查询的序列号，即执行SQL语句过程中查询表的顺序。

  - id值全部相同，查询表的执行顺序为由上到下执行

    ![id值全部相同](C:\Users\Administrator\Desktop\md\MySQL\images\id相同.png)

    > 执行顺序为由上到下顺序执行，即先查询t1表，再查询t3表，最后查询t2表。

  - id值全不相同（例如嵌套子查询），id值依次递增，id值越大表示执行优先级越高，就越先被执行

    ![id值全不相同](C:\Users\Administrator\Desktop\md\MySQL\images\id值全不相同.png)

    > 执行顺序为先查询t3表，再查询t1表，最后查询t1表

  - id值部分相同，在所有组中id值越大，优先级越高也就越先执行。如果id值相同则为从上到下顺序执行

    ![id值部分相同](C:\Users\Administrator\Desktop\md\MySQL\images\id值部分相同.png)

    > DERIVED为衍生表，执行顺序为先执行t3表，再执行derived2表（t3表），最后执行t2表

- **select_type**

  查询的类型，主要用于区别普通查询，联合查询、子查询等复杂查询类型。

  常见的值有SIMPLE  PRIMARY  SUBQUERY  DERIVED  UNION  UNION RESULT
  
- **type**

  显示查询使用了何种类型，从最好到最差依次是：

  system > const > eq_ref > ref > range > index > all

  **system查询类型**

  > 表中只有一行记录（等于系统表），这是const查询类型的特例，平时不会出现

  **const查询类型**

  > 表示通过索引查询，一次就查询到了。例如通过主键索引查询/唯一索引查询

