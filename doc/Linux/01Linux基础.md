#### Vim编辑器三种模式

1. 正常模式
2. 编辑模式
3. 命令行模式

学会如何在三种模式间切换

#### Vim编辑器常用快捷键

1. 拷贝：yy 拷贝，p 粘贴（正常模式下可以使用该快捷键）

   > 5yy  拷贝当前行及其下面的4行

2. 删除：dd 删除（正常模式）

3. 查找：/ + 查找的关键词（命令行模式）

   > 在处于查找状态是，输入 n  可以查找下一个

4. 行首|行尾：gg 由当前行调到行首，G由当前行调到行尾（正常模式）

5. 撤销：编辑模式下输入hello，切换到正常模式输入 u ，即可撤销hello

#### 输出重定向

1. 覆盖

   ```shell
   ls -l > dic.txt  
   ```

2. 追加

   ```powershell
   ls -l >> dic.txt 
   ```

#### 常用命令

1. head

   head命令用于显示文件头部内容，默认显示前10行内容

   ```shell
   # 显示文件的前5行内容
   head -n 5 /etc/profile  
   ```

2. tail

   tail 指令用于显示文件的尾部内容，默认显示最后10行

   ```powershell
   # 基本用法
   tail -n 5 /etc/profile
   tail -f ../log.txt # 用于实时追踪显示文件的更新内容
   ```

3. find

   find指令从指定目录下递归遍历其各个子目录，将满足条件的文件或者目录的绝对路径显示出来

   参数说明：

   | 参数  | 功能                             |
   | ----- | -------------------------------- |
   | -name | 按照指定文件名或者目录名进行查找 |
   | -user | 查找属于指定用户的所有文件       |
   | -size | 按照指定文件大小进行查找         |

   ```shell
   find /home/user1 -name hello.txt	
   ```

4. locate

   locate指令可以快速定位文件位置，它事先利用文件系统将文件及其路径存放在数据库中。可以实现通过数据库快速定位文件路径。

   *第一次运行前必须使用updatedb命令创建locate数据库*

   ```shell
   [root@localhost home]# updatedb
   [root@localhost home]# locate cal.txt
   ```


5. grep过滤查找 | 管道符

   grep过滤查找，表示将前面指令的查找结果作为输入，再次进行过滤查找

   参数说明：

   | 参数 | 说明       |
   | ---- | ---------- |
   | -n   | 显示行号   |
   | -i   | 忽略大小写 |

   ```shell
   [root@localhost java]# cat test.java | grep -ni private
   5:	private static String getName(){
   8:	private static int name = 101;
   ```

   