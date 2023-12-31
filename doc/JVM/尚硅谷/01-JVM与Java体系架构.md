### 1. JVM与Java体系架构

**作用：**Java虚拟机就是二进制字节码的运行环境。

**特点：**

- 一次编译，到处运行
- 自动内存管理
- 自动垃圾回收

#### 1.1 JVM整体结构

<img src="https://i.loli.net/2020/06/18/Kx7gNihmtcpWUOB.png" alt="JVM整体结构" style="zoom:67%;" />

#### 1.2 JVM指令集架构

**基于栈的指令集架构**

- 设计和实现方式更加简单

  > 程序的运行的本质是方法的执行。方法执行之前入栈，方法执行结束出栈，这种实现方式更加简单。

- 适用于资源受限的系统

- 指令流中的指令大部分是零地址指令，指令集更小，编译更加容易实现

  > 一地址指令：一个地址对应一个操作数
  >
  > 二地址指令：两个地址对应一个操作数
  >
  > 零地址指令：不需要存储操作数对应的地址

- 不需要硬件支持，支持跨平台操作

- **指令集小，但是指令多**

**基于寄存器的指令集架构**

- 典型应用：x86的二进制指令集（pc、android）
- 指令集架构完全依赖硬件，可移植性差
- 性能更好，操作更加高效
- 指令大部分是一地址指令、二地址指令、三地址指令
- 指令集大，但是指令少

#### 1.3 常用命令

- 反编译

  ```shell
  javap -v className.class
  ```

- 查看正在运行的进程

  ```shell
  jps
  ```

#### 1.4 Java虚拟机生命周期

1. 启动

   **java虚拟机的启动是通过类加载器（Bootstrap class loader）创建一个初始类（initial class）完成的，这个初始类是由虚拟机的具体实现来指定的**

2. 执行

   程序开始执行，java虚拟机就开始执行。

   **执行一个java程序，本质上是一个JVM进程的执行**

3. 结束

   程序正常执行退出

   程序遇到异常或者错误而异常终止退出

   由于操作系统出现错误而导致java虚拟机进程终止

   某线程调用Runtime.exit()方法或者System.exit()方法导致程序退出

#### 1.5 JVM发展历程

1. SUN Classic VM
2. Exact VM
3. HotSpot VM（现在用的）
4. IBM J9 VM