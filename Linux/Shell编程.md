#### Shell编程

> Shell 是一个用 C 语言编写的程序，它是用户使用 Linux 的桥梁。Shell 既是一种命令语言，又是一种程序设计语言。
>
> Shell 是指一种应用程序，这个应用程序提供了一个界面，用户通过这个界面访问操作系统内核的服务。

##### 运行shell脚本

```shell
[root@andre zhaojingchao]# vim test.sh
#!/bin/bash
echo "hello world!"

[root@andre zhaojingchao]# chmod +x test.sh  #添加可执行权限
[root@andre zhaojingchao]# ./test.sh   # 执行shell脚本，注意这里不能直接用test.sh
```

##### shell变量

 定义变量时，变量名不加美元符号（$，PHP语言中变量需要） 

```shell
your_name="runoob.com"
```

>  注意，变量名和等号之间不能有空格，这可能和你熟悉的所有编程语言都不一样。同时，变量名的命名须遵循如下规则： 
>
> -  	命名只能使用英文字母，数字和下划线，首个字符不能以数字开头。
> -  	中间不能有空格，可以使用下划线（_）。
> -  	不能使用标点符号。
> -  	不能使用bash里的关键字（可用help命令查看保留关键字）。

使用shell变量

```shell
your_name="qinjx"
echo $your_name
echo ${your_name}  #推荐使用此种{}方式
```

