### 1.Docker和传统虚拟机区别

- 传统虚拟机是虚拟出一套硬件系统，运行一个完整的操作系统，然后再系统上安装和运行软件
- docker容器不需要自己的内核，因此非常轻量级
- docker容器内都有一个属于自己的文件系统，每个容器之间互相隔离，之间互不影响

### 2.Docker的DevOps思想（开发、运维）

- 更快速地交付和部署
  - 传统部署：需要提供一堆配置文件、部署文档
  - docker：打包镜像，测试发布，一键运行
- 更便捷的升级和扩缩容
  - 一键即可在集群上打包 部署
- 更简单的系统运维
- 更高效的利用计算资源
  - 通过容器技术可以在一个服务器上部署很多个tomcat，将服务器的性能发挥到极致

### 3.快速开始

**docker架构图**

<img src="https://i.loli.net/2020/08/01/9VSxDTPkYJ5n6eZ.png" alt="docker架构图" style="zoom:100%;" />

#### 镜像（image）

#### 容器（container）

​	启动的镜像叫做容器，可以把容器理解为一个简易的linux系统

#### 仓库（repository）

​	仓库是存放镜像的地方，仓库分为共有仓库（官方的为docker hub，国内的镜像仓库有阿里云、网易等等仓库）、私有仓库（自己搭建的私有镜像仓库）

### 4.安装Docker

1. 环境

   centOS7

2. 内核版本

   ```shell
   [root@Andre /]# uname -r
   4.4.0-18362-Microsoft
   ```

3. 系统版本

   ```shell
   [root@Andre /]# cat /etc/os-release
   NAME="CentOS Linux"
   VERSION="7 (Core)"
   ID="centos"
   ID_LIKE="rhel fedora"
   VERSION_ID="7"
   PRETTY_NAME="CentOS Linux 7 (Core)"
   ANSI_COLOR="0;31"
   CPE_NAME="cpe:/o:centos:centos:7"
   HOME_URL="https://www.centos.org/"
   BUG_REPORT_URL="https://bugs.centos.org/"
   
   CENTOS_MANTISBT_PROJECT="CentOS-7"
   CENTOS_MANTISBT_PROJECT_VERSION="7"
   REDHAT_SUPPORT_PRODUCT="centos"
   REDHAT_SUPPORT_PRODUCT_VERSION="7"
   ```

4. 安装Docker，[参考docker官方文档](https://docs.docker.com/engine/install/centos/)

   ```shell
   #1. 删除旧的版本
   yum remove docker \
                     docker-client \
                     docker-client-latest \
                     docker-common \
                     docker-latest \
                     docker-latest-logrotate \
                     docker-logrotate \
                     docker-engine
   
   # 2.安装yum工具
   yum install -y yum-utils
   
   # 3.配置镜像仓库地址
   # 官方镜像地址（不推荐配置这个）
   yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
   # 阿里云镜像地址（推荐配置）
   yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
   
   # 4.更新yum软件包索引
   yum makecache fast
   
   # 5.安装docker引擎
   yum install docker-ce docker-ce-cli containerd.io
   
   # 6.启动docker
   systemctl start docker
   
   # 7.查看docker版本
   docker version
   
   # 8.docker 输出hello world
   docker run hello-world
   
   # 9.查看当前安装的镜像
   [root@geek yum.repos.d]# docker images
   REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
   hello-world         latest              bf756fb1ae65        7 months ago        13.3kB
   
   ```


### 5.Docker为什么比VM快

1. Docker有着比VM更少的抽象层

2. Docker直接利用的是宿主机的内核，VM是先加载出一个Guest OS，然后运行在该内核上

   所以新建一个容器时docker直接运行在宿主机内核上，运行级别为秒级；虚拟机是需要重新加载一个Guest OS内核，运行级别为分钟级

   

