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

##### 双引号与单引号区别

##### 变量作用范围

1. 局部变量

2. 全局变量

   发布全局变量

   ```shell
   export x=123
   
   echo $x
   ```

   取消全局变量

   ```shell
   export -n x
   ```

#### 数值运算及处理

计算并且获取结果

```shell
x=123; y=456
expr $x \* $y
```

##### 算式替换

使用$[ ] 表达是可以实现算式替换

- 格式

  $[整数1 表达式 整数2]

- 特点：

  乘法操作无需转义字符、运算符两侧可以没有空格、引用变量可以没有$符号

- 注意事项：

  计算结果替换表达式本身，结合echo命令才可以输出到屏幕上

```shell
x=123; y=456

echo $[x*y]
```

##### 随机整数

```shell
# 随机生成一个0 - 32676之间的一个整数
root@ansible ~]# echo $RANDOM
17646

# 限制随机数区间
[root@ansible ~]# echo $[RANDOM%1000+1]
645
```

##### 整数序列

使用seq命令可以根据指定条件输出一组整数（起始值缺省为1、步长缺省为1）

**seq命令格式**

- seq 结束值
- seq 起始值 结束值
- seq 起始值 步长 结束值

```shell
# 可以使用-s 来替换默认的换行分隔符
[root@ansible ~]# seq -s " " 10
1 2 3 4 5 6 7 8 9 10

# -w 显示等宽效果
[root@ansible ~]# seq -w 8 10
08
09
10

# 指定步长
[root@ansible ~]# seq -s " " -w 0 50 500
000 050 100 150 200 250 300 350 400 450 500
```

##### bc计算器实现小数运算

```shell
[root@ansible ~]#  bc
bc 1.06.95
Copyright 1991-1994, 1997, 1998, 2000, 2004, 2006 Free Software Foundation, Inc.
This is free software with ABSOLUTELY NO WARRANTY.
For details type `warranty'.

199.00*4.53
901.47
scale=3
199.00*4.53
901.470
quit
```

##### bc计算器的免交互方式（利用管道符实现）

```shell
[root@ansible ~]# a=12.34
[root@ansible ~]# echo "$a"
12.34
[root@ansible ~]# echo "$a*45.6" | bc
562.70
```

```shell
[root@ansible ~]# A=12.34;B=12.43
[root@ansible ~]# echo "$A >= $B" | bc
0
[root@ansible ~]# echo "$A < $B" | bc
1
```

#### 字符串处理

##### 字符串截取

```shell
# expr substr $var1 起始位置 截取长度

[root@ansible ~]# var=CentOS6.5
[root@ansible ~]# expr substr $var 1 6
CentOS
```



```shell
# 命令输出 | cut -c 起始位置-结束位置
# 命令输出 | cut -d '分隔符' -f 字段编号

[root@ansible ~]# var=CentOS6.5
[root@ansible ~]# echo $var | cut -c 5-6
OS
[root@ansible ~]# echo $var | cut -d "t" -f2
OS6.5
```

##### 字符串替换

```shell
# 替换第一个字符串
${var/oldString/newString}
# 替换所有的字符串
${var//oldString/newString}
```

##### tr单个字母替换

```shell
# 命令输出 | tr 'abc' 'ABC' (将所以的单个字母a替换成A b替换成B c替换成C)
# 命令输出 | tr -d 'abc' (删除所有'abc'字符)
```

##### 字符串路径分割

```shell
# dirname 取目录名称
# basename 取文件名称
```

##### 使用随机字符串

常见的随机性工具

- 随机变量：RANDOM
- uuid：uudigen
- 特殊设备文件：head -1 /dev/urandom

```shell
echo $RANDOM | md5sum
head -1 /dev/urandom | md5sum | cut -8
```

##### 命令替换

1. 使用 ` 反撇号实现

   ```shell
   [maple@localhost /]$ which tr
   /usr/bin/tr
   [maple@localhost /]$ rpm -qf /usr/bin/tr
   coreutils-8.22-24.el7_9.2.x86_64
   [maple@localhost /]$ rpm -qf `which tr`
   coreutils-8.22-24.el7_9.2.x86_64
   ```

2. 使用 $() 表达式实现

   ```shell
   # 方便嵌套使用
   [maple@localhost /]$ echo $(which tr)
   /usr/bin/tr
   [maple@localhost /]$ rpm -qf  $(which tr)
   coreutils-8.22-24.el7_9.2.x86_64
   [maple@localhost /]$ rpm -qi $(rpm -qf $(which tr))   # 查看tr所属软件包的用意 以及描述
   Name        : coreutils
   Version     : 8.22
   Release     : 24.el7_9.2
   Architecture: x86_64
   Install Date: Wed 24 Feb 2021 04:58:59 PM CST
   Group       : System Environment/Base
   Size        : 14594210
   License     : GPLv3+
   Signature   : RSA/SHA256, Wed 18 Nov 2020 10:16:51 PM CST, Key ID 24c6a8a7f4a80eb5
   Source RPM  : coreutils-8.22-24.el7_9.2.src.rpm
   Build Date  : Tue 17 Nov 2020 06:24:59 AM CST
   Build Host  : x86-01.bsys.centos.org
   Relocations : (not relocatable)
   Packager    : CentOS BuildSystem <http://bugs.centos.org>
   Vendor      : CentOS
   URL         : http://www.gnu.org/software/coreutils/
   Summary     : A set of basic GNU tools commonly used in shell scripts
   Description :
   These are the GNU core utilities.  This package is the combination of
   the old GNU fileutils, sh-utils, and textutils packages.
   ```

#### 条件测试

> 脚本的识别能力
>
> - 前一条命令是否执行成功
> - 文件或者目录的读写状态
> - 数值大小
> - 字符串是否匹配

##### 返回状态值 $?

```shell
# 输出0 表示执行成功
[maple@localhost /]$ ls -ld /root  &> /dev/null; echo $?
0

# 输出非0 表示执行失败
[maple@localhost /]$ ls -ld /rootx  &> /dev/null; echo $?
2
```

##### 专用测试工具test

test 选项 文件/目录

| 选项 | 选项描述                                |
| ---- | --------------------------------------- |
| -e   | 检测对象是存在(Exist)，是 返回真        |
| -d   | 检测对象是目录(Directory)，是 返回真    |
| -f   | 检测对象是文件(File)，是 返回真         |
| -w   | 检测对象有可写权限(Write)，是 返回真    |
| -r   | 检测对象有可读权限(Read)，是 返回真     |
| -x   | 检测对象有可执行权限(eXcute)，是 返回真 |

```shell
[maple@localhost /]$ test -f /etc/hosts; echo $?
0
```

##### 字符串比较

| 测试选项 | 含义             |
| -------- | ---------------- |
| =        | 两个字符串相同   |
| !=       | 两个字符串不相同 |
| -z       | 字符串值相同     |
| -n       | 字符串值不相同   |

```shell
# 判断当前用户是否为root用户，是则输出
[maple@localhost /]$ [ $USER = 'root' ] && echo $USER

# 判断当前位置是否不在/tmp 目录
[maple@localhost /]$ [ $PWD != '/tmp' ] && echo $PWD
/
```

#### 流程控制

```shell
[root@hplap2104 lucas]# cat check_mount_dir.sh
#!/bin/bash
MOUNT_DIR="/media/cdrom"
if [ ! -d ${MOUNT_DIR} ]
then
  mkdir -p ${MOUNT_DIR}
fi
```

```shell
[root@hplap2104 lucas]# cat pinghost.sh
#!/bin/bash
ping -c 3 -i 0.2 -W 3 $1 &> /dev/null
if [ $? -eq 0 ]; then
  echo "host $1 is up"
else
  echo "host $1 is down"
fi
```

```shell
[root@hplap2104 lucas]# cat  grade.sh
#!/bin/bash
read -p "please input the grade(0-100): " FS

if [ $FS -ge 80 ] && [ $FS -le 100 ];then
  echo "$FS grade, youxiu"
elif [ $FS -ge 60 ] && [ $FS -le 79 ]; then
  echo "$FS grade, lianghao"
else
  echo "$FS buhege"
fi
```





































