### 建议不会的不要写简历里，做单个技术的深度，不要太广泛

### 计算机基础知识篇

#### Java基础

1. 抽象类和接口的区别
2. HashTable 如何实现线程安全？和ConcurrentHashMap有哪些区别？
3. java保证线程安全方式
4. 对java哪块比较熟，hashmap结构，怎么解决线程不安全的问题，自己设计一个线程安全的hashmap 。concurrenthashmap如果保证线程安全，扩容时锁住什么，锁住class还用调用实例对象方法吗
#### 计算机网络

##### [OSI七层网络模型](https://zhuanlan.zhihu.com/p/32059190)

![Snipaste_2022-06-09_07-57-50](https://pic1.zhimg.com/80/v2-2d62ba265be486cb94ab531912aa3b9c_720w.jpg)

| OSI模型    | 基本作用                         |
| ---------- | -------------------------------- |
| 应用层     | 为应用程序之间提供网络服务       |
| 表示层     | 数据格式化、加密、解密           |
| 会话层     | 建立、维护、管理会话连接         |
| 传输层     | 建立、维护、管理端到端之间的连接 |
| 网络层     | IP寻址和路由选择                 |
| 数据链路层 | 控制物理层和网络层之间的通信     |
| 物理层     | 比特流传输                       |

| TCP/IP 五层模型 | 常见协议                | 典型硬件设备 |
| --------------- | ----------------------- | ------------ |
| 应用层          | HTTP FTP DNS SMTP       | 计算机       |
| 传输层          | TCP UDP                 | 防火墙       |
| 网络层          | ICMP  IGMP  IP ARP RARP | 路由器       |
| 数据链路层      | 由底层网络定义的协议    | 交换机       |
| 物理层          | 由底层网络定义的协议    | 网卡         |

##### [TCP三次握手](https://hit-alibaba.github.io/interview/basic/network/TCP.html)

##### TCP如何保证可靠传输（三次握手，建立连接，滑动窗口，快重传等）

##### 在应用层可以监听传输层的数据包么

##### MTU最大多大，在哪一层

##### Session、Cookie、Token的区别

<font color="red">出现cookie和session的原因：由于HTTP协议是无状态的协议，所以如果服务端需要记录用的状态就需要通过某种机制来识别具体的用户。此时cookie和session就</font>

Session：我们把这种能识别哪个请求由哪个用户发起的机制称为 Session（会话机制），生成的能识别用户身份信息的字符串称为 **sessionId**，sessionId 需要借助 cookie 的传递才有意义。

[**Session的痛点**](https://www.zhihu.com/question/19786827/answer/2064471064)

在单点系统中session可以满足用户认证的需求，但是在分布式高可用系统中通过session机制来完成用户认证就会出现问题

![](https://i.bmp.ovh/imgs/2022/06/11/92377a4006b7901f.png)

如图示：客户端请求后，由[负载均衡器](https://www.zhihu.com/search?q=负载均衡器&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A2064471064})（如 Nginx）来决定到底打到哪台机器。假设登录请求打到了 A 机器，A 机器生成了 session 并在 cookie 里添加 sessionId 返回给了浏览器，那么问题来了：下次添加购物车时如果请求打到了 B 或者 C，由于 session 是在 A 机器生成的，此时的 B,C 是找不到 session 的，那么就会发生无法添加购物车的错误，就得重新登录了，此时请问该怎么办。主要有以下三种解决方案

1. session复制

   ![](https://pic1.zhimg.com/80/v2-75d3d54a13e5ff882f55f932db4e6249_720w.jpg?source=1940ef5c)

2. session粘连

   ![](https://pic2.zhimg.com/80/v2-36bbaea261ab09dc07ce0fc33fa851b6_720w.jpg?source=1940ef5c)

3. session共享

   ![](https://pic1.zhimg.com/80/v2-7c43c11f9679bb9267c72dfb028cada6_720w.jpg?source=1940ef5c)

**Session和Cookie本质上都是用来记录用户信息的。主要区别在于**

1. 认证过程的区别

   Cookie认证过程：服务端会给客户端颁发一个Cookie，浏览器会把Cookie保存起来。当浏览器再起请求网站时会把请求地址和Cookie一同发给服务器。服务器检查该cookie，从而判断用户的状态。

   Session认证过程：当浏览器访问服务器时，服务器会把客户端信息以某种形式记录在服务器上，这就是session。当下次浏览器再次访问服务器时，只需要从该session查找用户的状态即可。

2. 存储位置不同

   Cookie存储在客户端（浏览器中），Session存储在服务端。

3. 存储容量不同

   Cookie的存储容量很小，Session的存储容量很大

4. 安全性不同

   Cookie安全性较低，Session安全性很高

**Token**

![](https://pic1.zhimg.com/80/v2-feb6b5939996bf5c25adc5da9bc6c37f_720w.jpg?source=1940ef5c)

可以看到 token 主要由三部分组成 1. header：指定了签名算法 2. payload：可以指定用户 id，过期时间等非敏感数据 3. Signature: 签名，server 根据 header 知道它该用哪种签名算法，再用密钥根据此签名算法对 head + payload 生成签名，这样一个 token 就生成了。

##### http和https的区别

<font color="red">HTTP协议：</font>HTTP是一个基于TCP/IP通信协议来传递数据的协议

<font color="red">基于HTTP协议访问百度网站过程：</font>

<img src="https://i.bmp.ovh/imgs/2022/06/12/e1b327f68296f0c5.png" style="zoom:67%;" />

1. DNS域名解析服务器解析百度网站的IP地址
2. 发起TCP请求，三次握手建立连接
3. HTTP请求
4. HTTP响应
5. 客户端将请求到的数据和资源渲染给前端用户

<font color="red">HTTP协议特点：</font>

1. HTTP协议支持客户端/服务端模式，也是一种请求/响应模式的协议
2. 无连接。每次连接只能处理一个请求
3. 无状态。协议不知道是哪个用户发起的请求

<font color="red">HTTP协议存在的问题：</font>

1. 请求信息明文传输，容易被窃听截取
2. 数据的完整性未校验，容易被篡改
3. 没有验证对方身份，存在冒充危险

<font color="red">HTTPS协议</font>

一般理解HTTPS协议为：HTTP协议 + SSL/TLS协议，通过SSL证书来验证服务器的身份，并未浏览器和服务端之间的通信进行加密

<font color="red">HTTPS协议通信流程</font>

<img src="https://i.bmp.ovh/imgs/2022/06/12/8ff4554c50b47bf8.png" style="zoom: 50%;" />

<font color="red">HTTPS协议缺点</font>

1. HTTPS协议多次握手，导致页面的加载时间延长近50%；
2. HTTPS连接缓存不如HTTP高效，会增加数据开销和功耗；
3. 申请SSL证书需要钱，功能越强大的证书费用越高
4. SSL涉及到的安全算法会消耗 CPU 资源，对服务器资源消耗较大。

HTTP协议和HTTPS协议的区别

1. HTTPS协议需要到CA申请证书，一般证书都是收费的
2. HTTP协议是超文本传输协议，信息是明文传输的；HTTPS协议则是具有安全性的SSL加密传输协议，比HTTP协议更加安全
3. HTTP和HTTPS使用的是完全不同的连接方式，连接端口也不一样，HTTP协议默认端口是80；HTTPS协议默认端口是443

##### 非对称加密

##### 请求头？如何设置过期时间

##### 浏览器输入URL后整个响应过程

#### 操作系统

#### 并发编程（进度40%）

1. 线程池的核心参数
2. 线程池原理
3. 线程安全的概念，不安全的时候体现在哪里
4. synchronized和volatile的区别
5. threadlocal原理
6. 实现死锁（说思路）
7. 乐观锁和悲观锁性能区别
8. 锁升级的过程
#### JVM（进度35%）

1. JVM内存划分
#### 设计模式

1. 设计模式（单例，工厂，代理，建造者）
#### MySQL（进度50%）

1. MYSQL的索引优化，设计规则，以及一些SQL优化
2. mysql的事务隔离级别
3. MySQL回表
### 框架篇

#### Spring原理

1. Spring MVC请求过程
2. Spring原理
3. Spring有哪些好处
4. 过滤器和拦截器的区别？
5. bean加载的过程
6. spring怎么解决循环依赖的
7.  只说了三级缓存，然后问我每一级的作用
#### Netty源码

#### Redis

1. Redis缓存，本地缓存，MYSQL保证数据一致性
2. redis 集群什么是哨兵机制 击穿雪崩
3. 在[redis](https://www.nowcoder.com/jump/super-jump/word?word=redis)里面的存储结构是如何设计的，为什么要用hash ，大key问题怎么解决的，序列化和反序列化会遇到什么问题
4. 怎么实现[redis](https://www.nowcoder.com/jump/super-jump/word?word=redis)对接口的限流的，多个[redis](https://www.nowcoder.com/jump/super-jump/word?word=redis) 操作的原子性怎么保证的
#### Zookeeper

#### ES

1. es 增删数据，可能取不到的场景？
2. es可以放数据，mysql也可以放数据，你这里是如何划分的呢，依据是什么
### RPC框架

1. 讲一下rpc调用过程
2. HTTP请求和RPC请求的区别
#### Spring Cloud

### 消息队列篇

#### Kafka

1. Kafka原理
### Linux

1. Linux命令，权限，vim等
### 分布式篇

#### 分布式锁

1. 分布式锁如何实现
#### 分布式事务

#### 《分布式系统原理介绍》

#### 《DDIA》

1. 登录凭证分布式环境下怎么考虑的（放[redis](https://www.nowcoder.com/jump/super-jump/word?word=redis)）
### 数据结构与算法篇

#### 数据结构

1. 链表
2. 队列
3. 堆
4. 树
    1. 二叉查找树
    2. 完全二叉树
    3. B树
    4. B+树
    5. B-树
    6. 红黑树
#### 算法

1. 排序算法
2. 查找算法
3. 递归算法
4. 动态规划算法
5. 贪心算法
6. 分治算法
7. [二叉树](https://www.nowcoder.com/jump/super-jump/word?word=%E4%BA%8C%E5%8F%89%E6%A0%91)，左旋右旋（忘得一干二净，瞎说一通），说下b+树
8. 说几个[排序](https://www.nowcoder.com/jump/super-jump/word?word=%E6%8E%92%E5%BA%8F)[算法](https://www.nowcoder.com/jump/super-jump/word?word=%E7%AE%97%E6%B3%95)

### 常见场景解决方案篇

1. 项目中消息可靠性保证
2. 你如果用缓存的话，数据的同步一致怎么考虑的
3. 如何监控服务的各种状态指标，了解字节内部是如何做的吗，如果我想监测调用链路的响应时长，你怎么设计
4. jwt是怎么用的，包含哪些部分，jwe和jws 了解过吗
5. 输入url到网页显示过程
6. dns解析过程
7. 请求慢排查，（用户端，网络，服务器负载，jvm参数，mysql负载）
### 软件开发

1. 开发一个软件过程（软件工程）
### 开放性问题

1. 对Zoom的了解
2. 未来职业规划
3. 找工作关注公司哪些情况
4. 你最大的优点/缺点是什么？举例说明









<font color="red"></font>





