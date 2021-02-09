### K8S架构

<img src="https://i.loli.net/2021/02/09/1xEbeNvWTHVXcRn.png" alt="image-20210209101318597" style="zoom:67%;" />

##### 常用组件功能介绍

1. api server

   所有服务访问的统一入口

2. Controller Manager

   主要用作副本管理的，维持期望的副本数量

3. Scheduler

   负责介绍任务，选择分配合适的节点进行任务的分配

4. ETCD

   键值对数据库，持久化存储K8S集群中重要的数据

5. Kubelet

   直接和容器引擎进行交互，实现容器的声明周期的管理

6. Kube-proxy

   负责写入规则至IpTables、IpVS，实现服务的映射访问

7. CoreDNS

   负责给集群中的SVC提供一个域名/IP地址对应关系解析器

   也可以实现负载均衡

8. Dashboard

   提供了一个B/S架构的统一面板服务

9. Ingress Controller

   官方K8S只能实现四层代理，Ingress可以实现七层代理（可以根据主机名或者域名进行负载均衡）

10. Federation

     提供了一个可以跨集群中心多K8S管理的功能

11. Prometheus

    提供K8S集群监控功能

12. ELK

    提供K8S集群日志收集、监控、分析功能

##### Pod

1. 分类

   1. 自主式pod
   2. 控制器管理的pod

2. Relication Controller

   RC是用来确保容器中运行的副本数量始终大于用户定义的副本数量。如果有容器异常退出，k8s会自动创建新的pod来代替异常退出的pod；而如果多出来的异常容器也会被自动回收。

   在新版本的K8S中建议使用Relication Set代替Relication Controller’

3. Relication Set

   RS和RC没有本质区别，只是名字不一样而已，除此之外RS支持集合式selector

4. Deployment

   虽然RS可以独立使用返，但是建议使用Deployment来管理RS，这样无需担心和其他机制不兼容的问题

5. Horizontal Pod Autoscaling（HPA）

   平滑扩容即弹性伸缩。在V1版本中可以根据cpu利用率来进行扩缩容，在v alpha版本中可以根据内存和用户定义的metric进行扩缩容

6. Stateful Set

   是为了解决有状态服务的问题，对应deployment和RS为了无状态服务设计的。

   具体应用场景：

   1. 稳定的持久化存储

      pod重新调度后还是能够访问到相同的持久化数据

   2. 稳定的网络标识

      pod重新调度后其PodName和HostName不变

   3. 有序部署，有序扩展

      即pod是有顺序的，在部署或者扩展容器的时候要根据定义的顺序依次执行

      eg：先启动MySQL服务，再启动Tomcat服务，再启动NGINX服务

   4. 有序收缩，有序删除

##### 网络通讯模型

1. 同一个Pod内的多个容器之间通过IO方式实现通讯
2. 各个Pod之间通过overlay network实现通讯
3. pod与service之间的通讯：各个节点的IpTables规则







