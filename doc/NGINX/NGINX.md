## NGINX教程

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

### 3.NGINX配置实例

#### 3.1反向代理配置

#### 3.2负载均衡配置

#### 3.3动静分离配置

#### 3.4高可用集群配置

### 4.NGINX原理









