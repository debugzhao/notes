### 部署Nginx

```shell
#1.拉取镜像
[root@geek home]# docker pull nginx
Using default tag: latest
latest: Pulling from library/nginx
6ec8c9369e08: Already exists 
d3cb09a117e5: Pull complete 
7ef2f1459687: Pull complete 
e4d1bf8c9482: Pull complete 
795301d236d7: Pull complete 
Digest: sha256:0e188877aa60537d1a1c6484b8c3929cfe09988145327ee47e8e91ddf6f76f5c
Status: Downloaded newer image for nginx:latest
docker.io/library/nginx:latest

#2.查看所有镜像
[root@geek home]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
mysql               latest              e3fcc9e1cc04        10 days ago         544MB
nginx               latest              8cf1bfb43ff5        11 days ago         132MB
centos              latest              831691599b88        6 weeks ago         215MB


#3.运行nginx容器
[root@geek home]# docker run -d --name nginx01 -p 3344:80 nginx
04465ba331d78600a3798d29f856f6fc7e682ffb5cc2607f4e36550eb38d3b9e
#参数说明
-d 以后台方式运行
--name 给镜像起别名
-p 端口映射规则 3344位主机暴露的端口号，80为nginx端口号，通过主机ip:3344就可以访问nginx容器服务

#4.查看进程
[root@geek home]# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                  NAMES
04465ba331d7        nginx               "/docker-entrypoint.…"   7 seconds ago       Up 6 seconds        0.0.0.0:3344->80/tcp   nginx01

#5.在主机上访问暴露的3344端口
[root@geek home]# curl localhost:3344
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>

#6.进入Nginx容器
[root@geek home]# docker exec -it nginx01 /bin/bash
root@04465ba331d7:/# whereis nginx
nginx: /usr/sbin/nginx /usr/lib/nginx /etc/nginx /usr/share/nginx

```

**外网访问NGINX**

![Nginx](https://i.loli.net/2020/08/02/3rcFD4mV5hWgKXU.png)

### 部署Tomcat

```shell
#下载镜像
[root@geek home]# docker pull tomcat:9.0
#启动tomca服务
[root@geek home]# docker run -it --name tomcat01 -p 8081:8080 tomcat:9.0
```

**访问tomcat**

![tomcat401](https://i.loli.net/2020/08/02/SiN7eY5QPxU9dGO.png)

外网访问tomcat发现，返回404，原因：由于阿里云镜像的原因，默认下载的是最小的镜像，所以一会出现部分命令不支持、文件没有的情况。出现404就是疑问tomcat服务的webapps目录没有必要的文件。

解决方式：复制webapps.list目录中的文件到webapps目录中



