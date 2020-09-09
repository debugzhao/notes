### 版本控制是什么

Git 是一个开源的分布式版本控制系统，用于敏捷高效地处理任何或小或大的项目。

### 版本控制技术产生背景

为什么要使用版本控制？有什么应用场景？

- 备份文件
- 历史记录
- 版本回退
- 多段共享

### 主流版本控制技术

#### SVN	

> SVN是集中式版本控制系统，版本库是集中放在中央服务器。工作时，首先要从中央服务器哪里得到最新的版本（可以理解为代码），团队协作开发后需要把自己的代码推送到中央服务器。
>
> 集中式版本控制系统是必须联网才能工作，如果在局域网还可以，带宽够大，速度够快。但是不在局域网下就会出现网速很慢的情况。

#### Git

> Git是分布式版本控制系统，它就没有中央服务器，每个人的电脑就是一个完整的版本库。
>
> 如何进行多人协作？比如说用户A在电脑上修改了Demo.java文件，其他人也在电脑上修改了Demo.java文件，这时只需把各自的修改文件推送**（push）**到远程仓库，然后再拉取**（pull）**远程仓库内别人的提交记录。就可以所有人的修改记录

### Git原理/工作流程

![图片描述](https://img.mukewang.com/59c31e4400013bc911720340.png)

**概念介绍：**

- `Workspace`：工作区
- `index：`缓存区
- `Repository：`本地仓库
- `Remote:`远程仓库

**工作流程介绍**：

1. add
2. commit
3. push
4. fetch
5. pull
6. checkout

### Git安装

[下载地址](https://pc.qq.com/detail/13/detail_22693.html)

**基础配置**

```shell
//配置用户名
git config --global user.name ""
//配置邮箱账户
git config --global user.email ""
```

### Git常用命令

```shell
//创建仓库 初始化仓库
git init
//将改动或者创建的文件添加到暂存区
git add 文件名
//将暂存区的文件提交到远程仓库
git commit -m "提交描述"
//查询工作区内文件的提交状态
git status # 红色背景文件表示还未提交
//对比提交文件的改动地方
git diff 文件名
//查看提交日志
git log
//查看提交版本号
git reflog
//版本回退
git reset --hard HEAD 版本号
```

### 基于码云进行版本控制

1. 生成SSH key

   ```shell
   ssh-keygen -t rsa –C “youremail@example.com”
   ```

2. 添加公钥到码云、

3. 创建仓库

4. clone仓库

   ```shell
   git clone https://github.com/tugenhua0707/testgit.git
   ```

5. 创建并切换分支

   ```shell
   git checkout -b dev
   ```

6. 查看所有分支

   ```shell
   //当前所在分支前面会有个 *
   git branch
   ```

7. 合并指定分支到当前分支上

   ```shell
   git merge dev
   ```

8. 删除分支

   ```
   git branch -d dev
   ```













