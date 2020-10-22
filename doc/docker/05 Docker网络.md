### Docker0

查看虚拟机网卡信息：

![image-20201022221058693](https://i.loli.net/2020/10/22/d8lVoyzgqvrDRZa.png)

#### 思考

在容器内部可以ping通容器外部网络吗？两个容器之间可以互相ping通吗？**

```shell
# 运行tamcat镜像
docker run -d -P --name tomcat01 tomcat

# 查看tamcat容器内部网络地址
root@root:/etc/docker# docker exec -it tomcat01 ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
8: eth0@if9: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    # tomcat容器内部的ip地址
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever
       
# 思考：在容器外部能否ping通tomcat容器
root@root:/etc/docker# ping 172.17.0.2
PING 172.17.0.2 (172.17.0.2) 56(84) bytes of data.
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.056 ms
64 bytes from 172.17.0.2: icmp_seq=2 ttl=64 time=0.069 ms
64 bytes from 172.17.0.2: icmp_seq=3 ttl=64 time=0.064 ms
# 根据以上结果表示，在容器外部可以ping通容器内部(同一个网段下都能ping通)
```

#### 原理

只要安装了docker容器，linux就会有一个名字为docker0的网卡。我们只要启动一个docker容器，docker就会给相应的容器分配一个ip/创建一个网卡，使用的是桥接模式，本质上是用evth-pair技术实现的。

![image-20201022230356726](https://i.loli.net/2020/10/22/RsncaC1WZ5ieDJ9.png)