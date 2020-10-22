### Docker镜像原理

#### 镜像是什么

镜像是一种轻量级、可以独立执行的软件包，用来打包软件的运行环境和基于运行环境开发的软件。

它包含运行某个软件所需要的全部内容，包括：代码、运行时、库、环境变量、配置文件等

#### 镜像加载原理

- UnionFS联合文件系统

  UnionFS：联合文件系统是一种分层、轻量级并且高性能的文件系统。它支持对文件系统的修改作为一次提交来进行一层层叠加，同时可以将不同的目录挂载到同一个虚拟文件系统下。

  ​	联合文件系统是Docker镜像的基础，镜像可以通过分层进行继承，基于基础镜像（没有父镜像）可以制作不同的应用软件镜像

  **特点：**一次加载多个文件系统，但是从外面看来只能看到一个文件系统，联合加载会把各层的文件叠加起来，这样最终的文件会包含所需要的所有的底层文件和目录

- Docker镜像加载原理

  Docker镜像实际上是由一层一层文件系统构成，这种层级的文件系统称为UnionFS联合文件系统

  一个镜像例如：centOS由bootFS（boot file system）和 rootFS（root file system）构成

  - bootFS

    bootFS包括bootloader（引导加载器）和kernel（内核），引导加载器主要负责引导加载内核，Linux刚启动的时候会加载bootsFS文件系统，包括BootLoader和kernel。当BootLoader加载完成后整个系统就在内存中，这时内存的使用权由引导加载器转交给内核，此时系统也会卸载bootFS。

    Docker镜像最底层也是bootFS，这一点和Linux/Unix系统是一样的。

  - rootFS

    rootFS（root file system）包含的就是典型的Linux系统中的 /dev，/proc、/bin、/etc 等标准文件和目录，rootFS就是各种不同操作系统的发行版 例如Ubuntu、CentOS等

  ![rootFS](https://i.loli.net/2020/08/04/kYVFpBXO9mye7xR.png)
  
  对于一个精简的OS，rootFS占用容量可以很小，只需要包括基本的命令、工具和程序就可以了，因为底层用的是host的kernel，自己只需要提供rootFS。同时也由此可见不同的Linux发行版中rootFS基本是一致的，rootFS会有差别，所以不同的发行版可以共用一个bootfs

#### commit镜像

```shell
#查看docker commit帮助命令
[root@geek /]# docker commit --help

Usage:	docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]

Create a new image from a container's changes

Options:
  -a, --author string    Author (e.g., "John Hannibal Smith <hannibal@a-team.com>")
  -c, --change list      Apply Dockerfile instruction to the created image
  -m, --message string   Commit message
  -p, --pause            Pause container during commit (default true)
  
#自己提交一个镜像
[root@geek /]# docker commit -a "andre" -m "add webapps file" e1859edc8079 mytomcat:1.0
sha256:e37266651086eb88dc474a9cb41f96ce28827ea0976b4b939cd8a8d0127273a5

#查看自己提交的镜像
[root@geek /]# docker images
REPOSITORY            TAG                 IMAGE ID            CREATED             SIZE
mytomcat              1.0                 e37266651086        28 seconds ago      652MB
tomcat                9.0                 9a9ad4f631f8        9 days ago          647MB
tomcat                latest              9a9ad4f631f8        9 days ago          647MB
portainer/portainer   latest              62771b0b9b09        2 weeks ago         79.1MB
mysql                 latest              e3fcc9e1cc04        2 weeks ago         544MB
nginx                 latest              8cf1bfb43ff5        2 weeks ago         132MB
centos                latest              831691599b88        7 weeks ago         215MB
```

### 容器数据卷

#### 容器数据卷技术产生背景

当我们启动容器的时候会产生一定的数据，如果这些数据全部放在容器内，只要容器被删除那么这些数据也会随之丢失。

所以就会产生一个容器内与linux之间数据同步、持久化的需求，容器数据卷技术就实现了该需求。通过该技术我们可以实现容器之间，容器与linux之间的数据共享和持久化

**本质：**该技术本质是目录的挂载，将容器内的目录挂载linux目录下

![容器数据卷技术](https://i.loli.net/2020/08/08/6u8xUZTmQ3Kkj9L.png)

#### 使用容器数据卷

```shell
docker run -it -v /home/ceshi:/home --name myCentOS centos:latest

#参数说明
-v [-volume] 
/home/ceshi #主机目录
/home       #容器目录

#使用以上命令后可以实现主机目录 和 容器目录之间数据的互相同步
```

##### 查看挂载情况

```shell
docker inspect 容器id
```

![挂载情况](https://i.loli.net/2020/08/08/AbpLdHaRu6KXCUq.png)

#### MySQL实战

拉取MySQL 5.7版本镜像

```shell
docker pull mysql:5.7
```

启动MySQL镜像

```shell
docker run -d -p 3310:3306 -v /home/mysql/conf:/etc/mysql/conf.d -v /home/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 --name mysql01 mysql:5.7

#参数说明
-d    #后台启动
-p    #端口映射
-v    #目录挂载
-e    #配置MySQL root账户密码
--name#容器起别名
```

Navicat连接测试

<img src="https://i.loli.net/2020/08/08/BFNaH61pmt3zyVl.png" alt="Navicat" style="zoom:50%;" />

查看linux挂载目录中的文件

![查看linux 挂载目录中的文件](https://i.loli.net/2020/08/08/2qIW6mwZ3jFHXf5.png)

*删除启动的mysql容器，发现linux挂载目录中的文件依然存在，这就实现了数据的持久化操作。*

#### 具名挂载和匿名挂载

### Docker File

1. 概念

   docker file实际上是一个脚本文件，可以通过docker file实现自定义构建镜像

2. 常用参数

   ```shell
   FROM     		# 基础镜像，一切从这里开始
   MAINTAINER		# 镜像的维护者，姓名 + 邮箱
   RUN				# 镜像构建的时候需要运行的命令
   ADD				# 添加构建其他镜像
   WORDIR			# 镜像的工作目录
   VOLUME			# 镜像的挂载目录
   EXPOSE			# 保留端口配置
   CMD				# 指定这个容器启动的时候要运行的命令，只有最后一个会生效，可被代替
   ENTRYPOINT		# 指定这个容器启动的时候要运行的命令，可以追加命令
   ONBUILD			# 当构建一个被继承的DockerFile时，这个时候会运行ONBUILD指令
   COPY			# 类似ADD，将文件拷贝至镜像中
   ENV				# 构建的时候设置的环境变量	
   ```

3. #### 创建并编辑docker file文件

   ```shell
   FROM centos
   MAINTAINER zhaojingchao<andre215000@163.com>
   ENV MYPATH /usr/local
   WORKDIR $MYPATH
   
   RUN yum -y install vim
   RUN yum -y install net-tools
   
   EXPOSE 80
   
   CMD echo $MYPATH
   CMD echo "the centOS image have been built..."
   CMD echo /bin/bash
   ```

4. 通过docker file脚本构建镜像

   ```shell
   #注意该命令后面有个点
   docker build -f docker-file -t andre-centos:1.0 .
   
   #参数说明
   -f      docker file的文件名
   -t tag  镜像的名字和tag版本
   ```

5. 查看构建的镜像

   ```shell
   [root@geek docker-volume]# docker images
   REPOSITORY            TAG                 IMAGE ID            CREATED             SIZE
   andre-centos          1.0                 cb569a1a400f        11 seconds ago      215MB
   ```

#### 实战构建Tomcat镜像

```shell
FROM centos
MAINTAINER zhaojingchao<andre215000@163.com>
# 拷贝文件
COPY readme.txt /usr/local/readme.txt

# 添加压缩文件至 /usr/local/目录
ADD jdk-8u161-linux-x64.tar.gz /usr/local/
ADD apache-tomcat-8.5.55.tar.gz /usr/local/

# 执行安装vim命令
RUN yum -y install vim

# 配置环境变量目录
ENV MYPATH /usr/local
# 设置工作目录
WORKDIR $MYPATH

# 配置环境变量
ENV JAVA_HOME /usr/local/jdk1.8.0_161
ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
ENV CATALINA_HOME /usr/local/apache-tomcat-8.5.55
ENV CATALINA_BASH /usr/local/apache-tomcat-8.5.55
ENV PATH $PATH:$JAVA_HOME/bin:$CATALINA_HOME/lib:$CATALINA_HOME/bin

# 暴露8080端口
EXPOSE 8080

# 构建镜像的时候执行该命令
CMD /usr/local/apach-tomcat-8.5.55/bin/startup.sh && tail -f /usr/local/apach-tomcat-8.5.55/bin/logs/catalina.out
```

### 发布镜像

#### 发布镜像到Docker Hub

1. 登录docker hub

   ```shell
   root@root:/# docker login -u geek20
   ```

2. 发布镜像

   ```shell
   # 相关命令： docker push 仓库名称/镜像名:TAG
   
   root@root:/# docker push myentos:0.1
   The push refers to repository [docker.io/library/myentos]
   cd0919d221f7: Preparing 
   60ef431ad130: Preparing 
   291f6e44771a: Preparing 
   denied: requested access to the resource is denied   # 在这里显示被拒绝，原因：发布的镜像一定要指定仓库名称，发布镜像的tag
   
   # 解决方案：为镜像指定tag信息
   docker tag c175571689a2 geek20/mycentos:1.0
   
   # 然后重新发布镜像
   docker push  geek20/mycentos:1.0
   ```

#### 发布镜像到阿里云

1. 创建命名空间，仓库

   ![image-20201022114249296](https://i.loli.net/2020/10/22/qUpkJitzmEvgsKu.png)

2. 发布镜像步骤

   ![image-20201022114713562](https://i.loli.net/2020/10/22/sZ3PL1li7dIY5NB.png)