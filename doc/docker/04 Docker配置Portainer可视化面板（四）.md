#### 可视化面板

- portainer（不常用）
- rancher（CI/CD时可以选择该工具）

#### portainer快速开始

- 作用：

  portainer是Docker的图形化管理工具，提供一个后台管理页面来供我们操作镜像和容器

- 安装并启动命令

  ```shell
  ocker run -d -p 8088:9000 --restart=always -v /var/run/docker.sock:/var/run/docker.sock --privileged=true portainer/portainer
  
  #参数说明
  --restart    重启方式
  -p           端口映射
  -v           挂载目录的映射关系
  --privileged 授权
  ```

- 访问

  访问URL：ip + 8088

  - 第一次加载比较慢

  - 注册

  - 基本信息

    

