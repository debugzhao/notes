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

#### 3.3动静分离配置

#### 3.4高可用集群配置

### 4.NGINX原理









