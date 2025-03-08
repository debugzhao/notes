### Docker常用镜像命令

1. 帮助命令

   ```shell
   docker version
   docker info
   docker 命令 --help
   ```

2. [官方文档](https://docs.docker.com/engine/reference/commandline/kill/)

3. docker images 查看所有镜像

   ```shell
   [root@geek /]# docker images
   REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
   centos              latest              831691599b88        6 weeks ago         215MB
   
   #字段说明
   REPOSITORY  镜像仓库名称
   TAG         镜像tag(版本)
   IMAGE ID    镜像id
   CREATED     镜像创建时间
   SIZE        镜像大小
   
   #其他参数
   Options:
     -a, --all             列出所有的镜像
     -q, --quiet           只显示镜像ID
   ```

4. docker search 搜索指定镜像

   ```shell
   [root@geek /]# docker search mysql
   NAME                              DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
   mysql                             MySQL is a widely used, open-source relation…   9790                [OK]                
   mariadb                           MariaDB is a community-developed fork of MyS…   3572                [OK]   
   
   #其他参数
   --filter=STARS=3000 搜索stars数大于3000的镜像
   ```

5. docker pull下载指定镜像

   ``` shell
   [root@geek /]# docker pull mysql
   Using default tag: latest   #没有指定下载版本则默认下载最新的版本
   	latest: Pulling from library/mysql
   6ec8c9369e08: Pull complete 
   177e5de89054: Pull complete 
   ab6ccb86eb40: Pull complete 
   e1ee78841235: Pull complete 
   09cd86ccee56: Pull complete 
   78bea0594a44: Pull complete 
   caf5f529ae89: Pull complete 
   cf0fc09f046d: Pull complete 
   4ccd5b05a8f6: Pull complete 
   76d29d8de5d4: Pull complete 
   8077a91f5d16: Pull complete 
   922753e827ec: Pull complete 
   Digest: sha256:fb6a6a26111ba75f9e8487db639bc5721d4431beba4cd668a4e922b8f8b14acc #签名
   Status: Downloaded newer image for mysql:latest
   docker.io/library/mysql:latest #真实下载命令
   
   # 指定版本下载
   docker pull mysql:5.7
   ```

6. 删除镜像

   ```shell
   #通过指定镜像id进行删除
   docker rmi -f 镜像id
   #删除所有镜像
   docker rmi -f $(docker images -aq)
   ```

### Docker常用容器命令

*说明：运行着的镜像才可以被叫做容器*

**案例：**用docker下载并运行centOS

1. 下载容器

   ```shell
   root@geek /]# docker pull centos
   ```

2. 新建容器并启动

   ```shell
   docker run [可选参数] image
   
   #参数说明
   --name="image name"		#启动后容器的名字
   -d 						#后台方式运行
   -it						#交互方式运行，进入容器查看内容
   -p						#指定容器的端口号 
   	-p ip:主机端口:容器端口
   	-p 主机端口:容器端口(常用)
   	-p 容器端口
   -P（大写）				 #随机指定端口号
   ```

3. 启动并进入容器

   ```shell
   [root@geek /]# docker run -it centos /bin/bash
   [root@c76e5fb9a6e3 /]# ls  #查看容器内centOS目录(很多命令不完善)
   bin  dev  etc  home  lib  lib64  lost+found  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
   [root@c76e5fb9a6e3 /]# exit #从容器内退出
   exit
   [root@geek /]# 
   
   ```

4. 查看运行中的容器

   ```shell
   [root@geek /]# docker ps
   CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
   ```

5. 查看运行中的、运行过的容器

   ```shell
   [root@geek /]# docker ps -a
   CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                      PORTS               NAMES
   c76e5fb9a6e3        centos              "/bin/bash"         2 minutes ago       Exited (0) 2 minutes ago                        angry_villani
   8209c7682e1b        bf756fb1ae65        "/hello"            58 minutes ago      Exited (0) 58 minutes ago                       vigilant_herschel
   df9fbaae121a        bf756fb1ae65        "/hello"            9 hours ago         Exited (0) 9 hours ago                          suspicious_feynman
   ```

6. 退出容器

   ```shell
   exit #容器停止并退出
   ctrl + P + Q #容器不停止退出
   ```

7. 删除容器

   ```shell
   docker rm 容器id #删除指定的容器，不能删除正在运行的容器，使用-f(force)参数可以强制删除正在运行的容器
   docker rm -f $(docker ps -aq) #强制删除所有容器
   docker ps -aq | xargs docker rm #利用管道符删除所有的容器
   ```

8. 启动和停止容器

   ```shell
   docker start 容器id    #启动容器
   docker restart 容器id  #重启容器
   docker stop 容器id     #停止当前正在运行的容器
   docker kill 容器id     #强制停止当前容器
   ```

### Docker常用的其他命令

1. 后台启动容器

   ```shell
   docker run -d 镜像名
   #问题：docker ps后，发现centOS停止了。原因：如果把docker容器作为后台服务运行，则必须有一个对应的前台进程，如果docker发现没有对应的前台进程则会自动停止当前容器
   ```

2. 查看日志

   ```shell
   docker logs -ft --tail num 容器名称
   ```

3. 查看容器内进程信息

   ```shell
   [root@geek /]# docker top e9eb73ec3c8e
   UID                 PID                 PPID                C                   STIME               TTY                 TIME                CMD
   root                3401                3362                0                   15:23               pts/0               00:00:00            /bin/bash
   ```

4. 查看容器的元数据

   ```powershell
   docker inspect 容器id
   ```

5. 进入容器

   ```shell
   docker exec -it 容器id /bin/bash #进入容器内重新开启一个新的终端，可以在里面操作(常用)
   docker attach 容器id /bin/bash #进入容器内当前正在执行的终端
   ```

6. 从容器内拷贝文件到主机中

   ```shell
   #运行centos镜像
   [root@geek /]# docker run -it centos /bin/bash
   #切换目录，创建Test.java文件
   [root@a47842fa991f /]# cd home
   [root@a47842fa991f home]# touch Test.java
   [root@a47842fa991f home]# ls
   Test.java
   #退出容器
   [root@a47842fa991f home]# exit
   exit
   [root@geek /]# docker ps
   CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
   #查看所有运行过的容器信息
   [root@geek /]# docker ps -a
   CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                     PORTS               NAMES
   a47842fa991f        centos              "/bin/bash"         42 seconds ago      Exited (0) 8 seconds ago                       sleepy_keldysh
   #将Test.java文件从容器内拷贝至主机上
   [root@geek /]# docker cp a47842fa991f:/home/Test.java /home
   [root@geek /]# cd home
   [root@geek home]# ls
   Test.java
   ```

### Docker官方命令示意图





