### Docker0

查看虚拟机网卡信息：

![image-20201022221058693](https://i.loli.net/2020/10/22/d8lVoyzgqvrDRZa.png)

#### 思考

在容器内部可以ping通容器外部网络吗？两个容器之间可以互相ping通吗？

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

![image-20201024094644748](https://i.loli.net/2020/10/24/hZk84T96KHEXzex.png) 

我们发现docker容器创建的网卡都是成对出现的，`evth-pair`就是一对虚拟设备接口，一端连着docker容器，一端连着docker0网卡。因此`evth-pair`充当了虚拟桥梁的角色。

<img src="https://i.loli.net/2020/10/24/p9OrUkGxPYAbas5.png" alt="image-20201024095616122"  />

docker容器中所有的网络接口都是虚拟的（虚拟网络接口转发效率高）

### 自定义网络

#### 查看docker中所有的网络

```shell
root@root:/# docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
04ba9e12bfc5        bridge              bridge              local
a3c6b0888f6d        host                host                local
2089b784bbed        none                null                local
```

| 网络模式  | 描述                             |
| --------- | -------------------------------- |
| bridge    | 桥接模式（docker模式）           |
| none      | 不配置网络                       |
| host      | 和宿主机共享网络                 |
| container | 容器网络连通（局限性大，用得少） |

#### 测试自定义网络

```shell
# 我们在不指定网卡类型的情况下直接启动容器，是用的 bridge 桥接模式(docker0网卡指定的网络模式)
root@root:/# docker run -d -P --name tomcat01 tomcat
root@root:/# docker run -d -P --name tomcat01 --net bridge tomcat

# docker0网卡特点：默认网络模式；不能直接通过容器名称访问(通过--link参数可以解决)
```

#### 自定义网络

1. 自定义网络

   ```shell
   # --driver bridge 网络驱动(桥接模式)
   # --subnet 192.168.0.0/16 子网范围
   # --gateway 192.168.0.1 网关
   # mynet 自定义网络名称
   root@root:/# docker network create --driver bridge --subnet 192.168.0.0/16 --gateway 192.168.0.1 mynet
   0b9f026a5eabdf5993d05e635cbca0ad6e4fe6e90a342ac83b63ef73d2002704
   ```

2. 查看自定义网络

   ```shell
   root@root:/# docker network ls
   NETWORK ID          NAME                DRIVER              SCOPE
   04ba9e12bfc5        bridge              bridge              local
   a3c6b0888f6d        host                host                local
   0b9f026a5eab        mynet               bridge              local
   2089b784bbed        none                null                local
   ```

3. 查看自定义网络的内部信息

   <img src="https://i.loli.net/2020/10/25/NJmvBSlEiZzdwY9.png" alt="image-20201025153852210" style="zoom:80%;" />

4. 通过自定义的网络驱动启动容器（可以通过容器的名字实现容器间内部通信）

   ```shell
   root@root:/# docker run -d -P --name tomcat-mynet-01 --net mynet tomcat
   bbcdc2a22d35fec662dadf5b719dc12a6a10da282afebd154c2a611006463009
   root@root:/# docker run -d -P --name tomcat-mynet-02 --net mynet tomcat
   0a7d368e6ee83965d78097a5fabc69a05050ba41cd4db8ddbdbb02db97a3bc2b
   root@root:/# docker network inspect mynet
   [
       {
           "Name": "mynet",
           "Id": "0b9f026a5eabdf5993d05e635cbca0ad6e4fe6e90a342ac83b63ef73d2002704",
           "Created": "2020-10-25T15:30:10.491570703+08:00",
           "Scope": "local",
           "Driver": "bridge",
           "EnableIPv6": false,
           "IPAM": {
               "Driver": "default",
               "Options": {},
               "Config": [
                   {
                       "Subnet": "192.168.0.0/16",
                       "Gateway": "192.168.0.1"
                   }
               ]
           },
           "Internal": false,
           "Attachable": false,
           "Ingress": false,
           "ConfigFrom": {
               "Network": ""
           },
           "ConfigOnly": false,
           "Containers": {
               "0a7d368e6ee83965d78097a5fabc69a05050ba41cd4db8ddbdbb02db97a3bc2b": {
                   "Name": "tomcat-mynet-02",
                   "EndpointID": "a3085ec4d1ec8ae4a407eed0993b8de3b99758569932e0c643cd099afdcf906a",
                   "MacAddress": "02:42:c0:a8:00:03",
                   "IPv4Address": "192.168.0.3/16",
                   "IPv6Address": ""
               },
               "bbcdc2a22d35fec662dadf5b719dc12a6a10da282afebd154c2a611006463009": {
                   "Name": "tomcat-mynet-01",
                   "EndpointID": "5d87ddc9d51da52bd5677aee06de02cb4b732236a2a2f5af881603658f282db3",
                   "MacAddress": "02:42:c0:a8:00:02",
                   "IPv4Address": "192.168.0.2/16",
                   "IPv6Address": ""
               }
           },
           "Options": {},
           "Labels": {}
       }
   ]
   ```

5. 通过容器名称做ping测试

   ```shell
   # 通过ip地址可以实现容器间ping通
   root@root:/# docker exec -it tomcat-mynet-01 ping 192.168.0.3
   PING 192.168.0.3 (192.168.0.3) 56(84) bytes of data.
   64 bytes from 192.168.0.3: icmp_seq=1 ttl=64 time=0.095 ms
   64 bytes from 192.168.0.3: icmp_seq=2 ttl=64 time=0.065 ms
   64 bytes from 192.168.0.3: icmp_seq=3 ttl=64 time=0.066 ms
   --- 192.168.0.3 ping statistics ---
   3 packets transmitted, 3 received, 0% packet loss, time 41ms
   rtt min/avg/max/mdev = 0.065/0.075/0.095/0.015 ms
   
   # 直接通过容器名称也可以实现容器间ping通
   root@root:/# docker exec -it tomcat-mynet-01 ping tomcat-mynet-02
   PING tomcat-mynet-02 (192.168.0.3) 56(84) bytes of data.
   64 bytes from tomcat-mynet-02.mynet (192.168.0.3): icmp_seq=1 ttl=64 time=0.051 ms
   64 bytes from tomcat-mynet-02.mynet (192.168.0.3): icmp_seq=2 ttl=64 time=0.064 ms
   64 bytes from tomcat-mynet-02.mynet (192.168.0.3): icmp_seq=3 ttl=64 time=0.066 ms
   --- tomcat-mynet-02 ping statistics ---
   3 packets transmitted, 3 received, 0% packet loss, time 49ms
   rtt min/avg/max/mdev = 0.051/0.060/0.066/0.009 ms
   ```

6. 好处

   redis集群用一个网段、mysql集群用一个网段，不同的集群使用不同的网络，以保证集群的安全和健康

#### 不同网段之间实现网络连通

要求docker0网段连通mynet网段

<img src="https://i.loli.net/2020/10/25/YsaWzcnPOINFltG.png" alt="image-20201025162951919" style="zoom:80%;" />

1. 使用 `docker network connect`命令实现将 A子网中的容器加入到B子网中

   ```shell
   root@root:/# docker network connect mynet tomcat01
   ```

2. 查看`mynet`网络驱动中的容器信息

   ```shell
   root@root:/# docker network inspect mynet
   ```

   ![image-20201025164423686](https://i.loli.net/2020/10/25/CJ9ozDtaM6TWjVh.png)

3. ping通信测试

   ```shell
   root@root:/# docker exec -it tomcat01 ping tomcat-mynet-01
   PING tomcat-mynet-01 (192.168.0.2) 56(84) bytes of data.
   64 bytes from tomcat-mynet-01.mynet (192.168.0.2): icmp_seq=1 ttl=64 time=0.081 ms
   64 bytes from tomcat-mynet-01.mynet (192.168.0.2): icmp_seq=2 ttl=64 time=0.068 ms
   64 bytes from tomcat-mynet-01.mynet (192.168.0.2): icmp_seq=3 ttl=64 time=0.068 ms
   --- tomcat-mynet-01 ping statistics ---
   3 packets transmitted, 3 received, 0% packet loss, time 41ms
   rtt min/avg/max/mdev = 0.068/0.072/0.081/0.009 ms
   ```

### Redis集群部署实战

#### redis集群架构

![image-20201025164757492](https://i.loli.net/2020/10/25/NVCdrkAquzZ9QLP.png)

*模拟redis master3挂掉之后slave3替换master3成为主机*

#### 创建redis网卡

```shell
# 创建redis网卡
root@root:/# docker network create redis --subnet 172.16.0.0/16
fb5766c11cec5e41ccded6fb02acbbdf58bad6988ee04036964975eb0fc429d3

# 查看docker容器的网卡
root@root:/# docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
04ba9e12bfc5        bridge              bridge              local
a3c6b0888f6d        host                host                local
0b9f026a5eab        mynet               bridge              local
2089b784bbed        none                null                local
fb5766c11cec        redis               bridge              local
```

#### 创建redis集群的脚本

```shell
# 通过脚本创建6个redis配置
for port in $(seq 1 6) \
do \
mkdir -p /mydata/redis/node-${port}/conf
touch /mydata/redis/node-${port}/conf/redis.conf
cat << EOF >/mydata/redis/node-${port}/conf/redis.conf
port 6379
bind 0.0.0.0
cluster-enabledro yes
cluster-config-file nodes.conf
cluster-node-timeout 5000
cluster-announce-ip 172.16.0.1${port}
cluster-announce-port 6379
cluster-announce-bus-port 16379
appendonly yes
EOF
done
```

```shell
# 启动redis容器
docker run -p 6376:6379 -p 16376:16379 --name redis-6 \
-v /mydata/redis/node-6/data:/data \
-v /mydata/redis/node-6/conf/redis.conf:/etc/redis/redis.conf \
-d --net redis --ip 172.16.0.16 redis:5.0.9-alpine3.11 redis-server /etc/redis/redis.conf
```

```shell
# 创建集群
redis-cli --cluster create 172.16.0.11:6379 172.16.0.12:6379 172.16.0.13:6379 172.16.0.14:6379 172.16.0.15:6379 172.16.0.16:6379
```
















