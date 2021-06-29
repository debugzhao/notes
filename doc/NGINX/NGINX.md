## 900NGINX教程

### 1.基本概念

#### 1.1NGINX是什么？应用场景？

Nginx是一个高性能的HTTP和反向代理web服务器，特点是占用内存少，并发能力强。Nginx的静态处理能力很强，但是动态处理能力不足，因此，在企业中常用动静分离技术

#### 1.2反向代理

##### 正向代理

如果把局域网外的internet看做一个巨大的资源库，局域网中的客户端想要访问Internet，需要先经过代理服务器进行访问，那么这种代理服务器就是正向代理

<img src="https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/正向代理.347llwkxub20.png" alt="正向代理" style="zoom:50%;" />

##### 反向代理

其实客户端对代理是无感知的，因为客户端不需要任何的配置就可以访问。我们只需要将请求发送到反向代理服务器，**由反向代理服务器去选择目标服务器，获取数据之后再返回给客户端**

此时反向代理服务器和目标服务器对外来说就是一台服务器，**暴露的是代理服务器，隐藏了真实的服务器的IP地址**

<img src="https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/反向代理.75zp8ah589k.png" alt="反向代理" style="zoom: 50%;" />

#### 1.3负载均衡

单个服务器解决不了的性能瓶颈问题，我们可以增加服务器的数量，然后将请求分发到不同的服务器上。将原先请求集中到单个服务器的情况改为请求分发到不同的服务器上，这就是所谓的负载均衡

<img src="https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/负载均衡.6pzwwr6auco0.png" alt="负载均衡" style="zoom: 50%;" />



#### 1.4动静分离

为了加快网站的解析速度，可以把静态资源和动态资源由不同的服务器来解析，降低原来单个服务的压力

<img src="https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/动静分离.2zdbhcu32fo0.png" alt="动静分离" style="zoom: 50%;" />

### 2.NGINX安装、常用命令和配置文件

#### 2.1NGINX的安装

#### 2.2NGINX常用命令

```shell
# 查看版本
[root@localhost sbin]# ./nginx -v
nginx version: nginx/1.17.10

# 关闭
[root@localhost sbin]# ./nginx -s stop

# 重新加载NGINX配置文件
[root@localhost sbin]# ./nginx -s reload
```

#### 2.3NGINX配置文件

NGINX配置文件由三部分组成

##### 全局块

从配置文件开始到events内容，主要会配置NGINX服务器整体运行的指令

```shell
# worker_processes 值越大，可以支持的并发处理数量也越多
worker_processes  1;
```

##### events块

events块涉及的指令主要影响NGINX服务器与用户的网络连接

```shell
events {
    # 支持最大的连接数
    worker_connections  1024;
}
```

##### http块

这部分是NGINX服务器配置最频繁地部分。**代理**、**缓存**、**日志**定义等大部分功能以及**第三方模块**的配置都是在这里配的

注意http块包括**http全局块**和**server块**

1. http全局块

   http全局块配置包括文件引入、MIME-TYPE定义、日志自定义、连接超时时间、单链接请求数上线等等

2. server块

### 3.NGINX配置实例

#### 3.1反向代理配置1

![反向代理配置（tomcat）](https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/反向代理配置（tomcat）.31fk6t0pqpg0.png)

前提Tomcat服务配置好

1. 修改windows环境中该目录下hosts文件

   `C:\Windows\System32\drivers\etc`，打开`hosts文件`添加`172.20.18.164 www.tomcat.com`

2. 在NGINX中进行请求转发的配置（反向代理配置）

   <img src="https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/nginx配置（Tomcat）.61hyzt8cgp00.png" alt="nginx配置（Tomcat）" style="zoom:50%;" />

#### 3.1反向代理配置2

##### 实现效果

使用nginx反向代理，根据访问的路径不同跳转到不同的服务中。

```shell
# nginx监听端口为9001
# http:127:0.0.1:9001/edu --跳转--> 127.0.0.1:8080
# http:127:0.0.1:9001/vod --跳转--> 127.0.0.1:8081
```

##### 准备工作

1. 准备两台tomcat服务器，一个是8080端口，一个是9080端口
2. 分别创建文件夹和测试页面

##### 具体配置

<img src="https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/9001配置.69s0tb9hx5w0.png" alt="9001配置" style="zoom:67%;" />

```shell
    server {
        listen       9001;
        server_name 172.20.18.164;

        location ~ /edu/ {
            proxy_pass http://172.20.18.164:8080;
        }

        location ~ /vod/ {
            proxy_pass http://172.20.18.164:9080;
        }
    }
```

##### 测试

<img src="https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/8080server.2aemytoeqw00.png" alt="8080server" style="zoom:50%;" />

<img src="https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/9080sever.7krtx5kx34k0.png" alt="9080sever" style="zoom:50%;" />

##### location格式

location有两种格式：

1. 匹配uri类型，有四种参数可选，当然也可以不带参数

   ```shell
   location [ = | ~ | ~* | ^~ ] uri { ... }
   ```

2. 命名location，用@来标识，类似于定义goto语句块。

   ```shell
   location @name { ... }
   ```

location匹配参数解释

| 参数 | 解释                                                         |
| ---- | ------------------------------------------------------------ |
| 空   | location后没有参数直接跟着URI，表示前缀匹配，代表跟请求中的URI从头开始匹配。 |
| ~    | 执行一个正则匹配，区分大小写。                               |
| ~*   | 执行一个正则匹配，不区分大小写。                             |
| ^~   | 普通字符匹配，多用来匹配目录。                               |
| =    | 执行普通字符精确匹配。                                       |
| @    | "@" 定义一个命名的 location，@定义的locaiton名字一般用在内部定向<br/>例如error_page, try_files命令中。它的功能类似于编程中的goto。 |

#### 3.2负载均衡配置

##### 实现效果

浏览器输入http://172.20.18.164/edu/index.html，将请求分发到8080端口和9080端口，以实现负载均衡效果

##### NGINX配置

<img src="https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/负载均衡实战1.2o7hjlnt0l80.png" alt="负载均衡实战1" style="zoom:50%;" />

##### 负载均衡策略

1. 轮询策略

   每个请求按照时间注意分配到不同的后端服务器，如果服务器down掉，**NGINX可以自动剔除服务器**

   ```shell
   upstream myserver {
   	server 172.20.18.164:8080;
   	server 172.20.18.164:9080;
   }
   ```

2. weight加权策略

   weight表示权重，默认权重为1，权重越高流量被分发到的概率越大

   ```shell
   upstream myserver {
   	server 172.20.18.164:8080 weight=5;
   	server 172.20.18.164:9080 weight=10;
   }
   ```

3. ip_hash方式

   每个请求按照客户端的ip的hash结果进行分配，这样每个客户端都会固定访问到同一个后端服务器，可以**解决session共享的问题**

   ```shell
   upstream myserver {
   	ip_hash;
   	server 172.20.18.164:8080;
   	server 172.20.18.164:9080;
   }
   ```

4. fair方式（第三方）

   按照后端服务器的响应时间分配，**响应时间越短则优先分配**

   ```shell
   upstream myserver {    
   	server 172.20.18.164:8080;
   	server 172.20.18.164:9080;
   	fair;
   }
   ```

#### 3.3动静分离配置

  NGINX动静分离就是把动态请求和静态请求分开，可以理解为使用NGINX处理静态页面，tomcat处理动态页面。

动静分离从目前实现上讲大致分为两种：一种是纯粹地把静态文件独立成单独的域名，放在单独的服务器上，也是目前主流推崇的方案；另一种是把动态文件和静态文件混合分布，使用NGINX分隔。

通过使用`location`指定不同的后缀名实现不同请求的转发。通过使用`expires`参数设置，可以配置浏览器的缓存过期时间，减少与服务器之间的请求，**本质上是在浏览器端产生缓存**

<img src="https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/动静分离请求.ulxcyq7q4q8.png" alt="动静分离请求" style="zoom:50%;" />

##### 准备工作

1. 准备静态资源，用于访问

   ```shell
   [root@localhost data]# tree
   .
   ├── images
   │   └── test.png
   └── www
       └── test.html
   ```

2. 配置

   ```shell
       location /www/ {
           root  /data/;
           index  index.html index.htm;
       }
   
       location /image/ {
           root   /data/;
           autoindex on;
       }
   ```

#### 3.4高可用集群配置

1. 简单的负载均衡实现方案

   ![普通负载均衡](https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/普通负载均衡.35k8ox9hz8o0.png)

   多台tomcat服务器可以实现负载均衡效果，但是NGINX服务器挂掉，服务就不可用

2. 高可用集群方案架构实现

   <img src="https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/NGINX高可用效果.24rlscgbnwg0.png" alt="NGINX高可用效果" style="zoom:50%;" />

   配置多台NGINX服务器，其中一台作为**主NGINX服务器**，另外一台作为**从NGINX服务器**。 当主服务器宕机之后就把请求切换至从服务器，从服务器实现流量的分发。

   此外需要第三方软件`keepalived`实现检测主服务器/从服务器是否存活（脚本实现），对外需要提供一个虚拟IP访问的。主NGINX服务器宕机了就需要把虚拟ip**绑定**到从NGINX上，以上过程对用户来说是无感知的。

##### 配置高可用方案准备工作

1. 需要两台服务器，分别安装`nginx`和`keepalived`软件

   ```shell
   yum install keepalived -y
   ```

2. 安装keepalived 之后，在`/etc/keepalived`目录下有一个`keepalived.conf`文件

3. 完成高可用配置（主从配置）

   keepalived.conf配置如下：
   
   ```shell
   global_defs {	#全局定义
    notification_email {
    acassen@firewall.loc
    failover@firewall.loc
    sysadmin@firewall.loc
    }
    notification_email_from Alexandre.Cassen@firewall.loc
    smtp_server 192.168.6.129
    smtp_connect_timeout 30
    router_id LVS_DEVEL	#唯一不重复
   }
   
   #脚本配置
   vrrp_script chk_http_port {
    script "/usr/local/nginx/nginx_check.sh"	#检测脚本的路径及名称
    interval 2 #（检测脚本执行的间隔）
    weight -20	#权重。设置当前服务器的权重，此处的配置说明：当前服务器如果宕机了，那么该服务器的权重降低20
   }
   
   #虚拟IP配置
   vrrp_instance VI_1 {
    state BACKUP 	#主服务器写MASTER、备份服务器写BACKUP
    interface ens33 #网卡
    virtual_router_id 51 # 主、备机的 virtual_router_id 必须相同
    priority 90 	#主、备机取不同的优先级，主机值较大，备份机值较小
    advert_int 1	#时间间隔。每隔多少秒发送一次心跳检测服务器是否还活着，默认1秒发送一次心跳
    authentication {
    auth_type PASS
    auth_pass 1111
    }
    virtual_ipaddress {
    192.168.6.50 // VRRP H 虚拟IP地址,网段要和linux的网段一致，可以绑定多个虚拟ip
    } 
   }
   ```
   
   NGINX心跳检测脚本
   
   ```shell
   #!/bin/bash
   A=`ps -C nginx –no-header |wc -l`
   if [ $A -eq 0 ];then
    /usr/local/nginx/sbin/nginx
    sleep 2
    if [ `ps -C nginx --no-header |wc -l` -eq 0 ];then
    killall keepalived
    fi
   fi
   ```
   
   

### 4.NGINX原理







