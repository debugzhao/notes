#### Docker Compose

##### 概念

docker compose是一个通过`docker-compose.yml`配置文件来定义、管理容器集群的应用程序（批量容器编排）

compose是docker的一个开源项目，需要安装才能使用

##### 应用场景

部署一个web项目，涉及到Tomcat容器、MySQL容器、NGINX容器、Redis容器等等，可以通过docker compose来对多个容器进行编排，集中管理

##### 安装

1. 安装

   ```shell
   curl -L https://get.daocloud.io/docker/compose/releases/download/1.25.5/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
   ```

2. 授权

   ```shell
   sudo chmod 751 /usr/local/bin/docker-compose
   ```

3. 验证安装

   ```shell
   root@root:/usr/local/bin# docker-compose version
   docker-compose version 1.25.5, build 8a1c60f6
   docker-py version: 4.1.0
   CPython version: 3.7.5
   OpenSSL version: OpenSSL 1.1.0l  10 Sep 2019	
   ```

##### [快速开始（官方示例）](https://docs.docker.com/compose/gettingstarted/)

1. 必要文件

   <img src="https://i.loli.net/2020/11/01/oKkQtw1jpM7HTCy.png" alt="image-20201101182456804" style="zoom:80%;" />

2. 启动compose

   ```shell
   docker-compose up
   ```

3. 停止compose

   ```shell
   docker-compose down
   ```

4. 验证测试

   ![image-20201101181923832](https://i.loli.net/2020/11/01/cUO5KCq3uhWPef7.png)

![image-20201101182008959](https://i.loli.net/2020/11/01/RAuQW3La59InOmf.png)

**以前是通过`docker run`命令来逐个启动容器的，现在可以通过编写`docker-compose.yml`配置文件来一键启动/停止所有容器**

##### docker compose.yml编写规则

```yaml
# 共三层
# 版本层
version:
# 服务层
service:
	# web服务层
	web:
		# web层具体配置
	# redis服务层
	redis:
# 其他配置
# 数据卷配置、网络配置、镜像配置
```

##### [实战：搭建WordPress博客](https://docs.docker.com/compose/wordpress/)

##### 实战：搭建微服务项目

1. 创建微服务项目
2. 编写Dockerfile，构建镜像文件
3. 编写docker-compose.yml配置文件，实现容器编排

#### Docker Swarm

