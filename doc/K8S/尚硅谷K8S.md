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

