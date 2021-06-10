### Shell编程

#### 小技巧

1. 开启调试模式执行脚本

   ```shell
   # 创建添加账户的脚本文件
   [root@ansible shell]# cat add_user.sh
   #!/bin/bash
   useradd zhaojingchao
   
   passwd zhaojingchao
   
   # 通过调试方式执行脚本
   [root@ansible shell]# sh -x add_user.sh
   + useradd zhaojingchao
   useradd：用户“zhaojingchao”已存在
   + passwd zhaojingchao
   更改用户 zhaojingchao 的密码 。
   新的 密码：
   ```

2. 免交互方式执行脚本

   ```shell
   # 免交互方式执行脚本 (将echo输出的内容再传入给stdin)
   echo zhaojingchao1234 passwd --stdin zhaojingchao
   ```

3. 忽略无关输出（黑洞设备实现）

   **黑洞设备**：/dev/null

   1. 相当于这是一个只能写入数据，不能读数据的单向文件
   2. 存放到其中的数据会丢失
   3. 使用方法：&> /dev/null

   ```shell
   # 忽略无关输出
   echo zhaojingchao1234 passwd --stdin zhaojingchao $> /dev/null
   ```

4. 记录错误输出

   根据错误需要，可以将错误信息报错到指定文件

   ```shell
   [root@ansible dev]# useradd zhaojingchao
   useradd：用户“zhaojingchao”已存在
   
   [root@ansible dev]# useradd zhaojingchao 2> /tmp/error.log
   
   [root@ansible dev]# cat /tmp/error.log
   useradd：用户“zhaojingchao”已存在
   ```

5. 一个免交互更友好的脚本

   ```shell
   [root@ansible shell]# cat add_user.sh
   #!/bin/bash
   # create user
   useradd lucas 2> /tmp/err.log
   
   # set passwd
   echo 1234567 | passwd --stdin lucas &> /dev/null
   ```


#### 命令组合运用

##### 顺序分隔

使用分号，可以实现命令的顺序分隔

```shell
mkdir /newdir; cd /newdir
```

##### 逻辑与分隔

使用 && 符号可以实现逻辑与操作，期望所有的命令都能够执行成功，一旦前面的命令失败，后面的命令则不再执行

```shell
# 先编译再安装(一旦编译失败则不再执行后面的安装命令)
make && make install
```

##### 逻辑或分隔

使用 || 符号可以实现逻辑或分隔，输入的任何一条命令执行成功都符合预期

```shell
# 如果没有mikey用户，则创建用户
id mikey || useradd mikey
```

##### 简单的判断操作

```shell
id zhaojingchao &> /dev/null && echo yes || echo no
```

![image-20210610154659394](https://i.loli.net/2021/06/10/JUrq8zaxlDAp46G.png)

##### 管道操作

1. 管道操作符： | 

2. 管道操作的作用：将一端的命令输出，交给另一端的命令处理

3. 示例：

   分页查询所有的网络设备信息

   ```shell
   ifconfig | less
   ```

   计算/etc 目录下有多少个普通文件

   ```shell
   # find /etc -type f 使用递归方式列出etc目录下的所有普通文件
   # wc -l 计算上一个命令输出结果的个数
   find /etc -type f | wc -l
   ```

   统计正在处于监听状态的TCP端口的数量

   ```shell
   # 使用netsta命令列出所有的TCP连接信息
   # 将查找的结果交个 grep 进行过滤，-c 表示统计数量
   [root@ansible etc]# netstat -anpt | grep -c "LISTEN"
   10
   ```

   ##### 命令组合运用（标准输入/输出）

   | 类型         | 设备文件    | 文件描述符 | 默认设备 |
   | ------------ | ----------- | ---------- | -------- |
   | 标准输入     | /dev/stdin  | 0          | 键盘     |
   | 标准输出     | /dev/stdout | 1          | 显示器   |
   | 标准错误输出 | /dev/stderr | 2          | 显示器   |

   ##### 重定向

   | 类型       | 操作符 | 用途                                         |
   | ---------- | ------ | -------------------------------------------- |
   | 重定向输入 | <      | 将文本输入来源由键盘更改为指定文件           |
   | 重定向输出 | >      | 将正确的日志信息报错到文件（会进行覆盖操作） |
   | 重定向输出 | >>     | 将正确的日志信息报错到文件（会进行追加操作） |
   | 重定向错误 | 2>     | 将错误的日志信息报错到文件（会进行覆盖操作） |
   | 重定向错误 | 2>>    | 将错误的日志信息报错到文件（会进行追加操作） |
   | 混合重定向 | &>     | 相当于 > 和 2>  会进行覆盖操作               |

   ##### 混合重定向

   将正确、错误日志分别重定向到不同的日志文件

   ```shell
   ls -ld /root /root1 > /var/stdin.log 2> /var/error.log
   ```

   将正确、错误日志统一重定向一个日志文件

   ```shell
   ls -ld /root /root1 &> /var/tmp.log
   ```

#### Shell变量

##### 环境变量

##### 参数位置变量

在执行脚本时提供的命令行参数

```shell
[root@ansible shell]# cat test.sh
#!/bin/bash

echo ${1} ${3}

[root@ansible shell]# ./test.sh 1 2 3
1 3
```

##### 预定义变量

用来保存脚本程序的执行信息

| 变量名 | 含义                                                         |
| ------ | ------------------------------------------------------------ |
| $0     | 当前所在的进程或者脚本名                                     |
| $$     | 当前运行进程的PID号                                          |
| $?     | 命令执行之后，返回的状态值。 0：表示正常 1或者其他：表示异常 |
| $#     | 已经加载的位置变量的个数                                     |
| $*     | 所有位置变量的值                                             |

#### 变量值以及控制范围

双引号与单引号区别

















