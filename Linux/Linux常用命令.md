#### [查看文件详情](https://www.cnblogs.com/renmengkai/p/9452883.html)

#####  **stat命令** 

> **stat命令**：文件/文件系统的详细信息显示。
>
> stat命令主要用于显示文件或文件系统的详细信息，该命令的语法格式如下：
>
> -f　　不显示文件本身的信息，显示文件所在文件系统的信息
>
> -L　　显示符号链接
>
> -t　　简洁模式，只显示摘要信息

##### wc命令

> **wc命令**用来计算数字。利用wc指令我们可以计算文件的Byte数、字数或是列数，若不指定文件名称，或是所给予的文件名为“-”，则wc指令会从标准输入设备读取数据。
>
> wc -c filename 参数-c表示统计字符, 因为一个字符一个字节, 所以这样得到字节数

```shell
kali@kali:/$ wc -c ~/java/jdk-8u60-linux-x64.tar.gz
181238643 /home/kali/java/jdk-8u60-linux-x64.tar.gz
```

#### 给文件添加权限

```shell
sudo chmod a+x 文件名
sudo chmod a+w 文件名
```

#### 安装.rpm文件

```shell
rpm -ivh MySQL-server-5.5.48-1.linux2.6.i386.rpm
## -i(install)v(输入日志)h(hashcode输出进度条)
```

