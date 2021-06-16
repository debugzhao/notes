### 进程与线程

#### 进程和线程

##### 进程

- 进程由`指令`和`数据`组成。一个进程的运行需要将指令加载进CPU、数据加载进内存。在进程运行过程中还需要用到磁盘、网络等设备。综上，进程是用来加载指令、管理内存、管理IO的程序集
- 当一个程序运行时，从磁盘加载这个程序的代码至内存，这时也就开启了一个进程
- 进程也可以理解为一个程序的实例。大部分程序可以运行多个进程实例（记事本、浏览器），当然有个程序只能运行一个进程实例（微信、网易云音乐）

##### 线程  

- 一个进程之内可以运行多个线程
- 一个线程就是一个指令流，将指令流中的一条条指令以一定的顺序交给CPU执行
- 线程是CPU调度的最小单位

##### 二者对比

- 进程之间相互独立。线程存在于进程的内部，是进程的一个子集
- 进程拥有共享资源，如内存空间，供其内部的线程共享使用
- 进程之间的通信比较复杂
  - 同一台计算机内的进程通信称为IPC（Inter Process Communication）通信
  - 不同计算机之间的进程通信，需要经过网络通信，并且遵守一定的协议（HTTP协议）
- 线程之间通信比较简单，因为它们共享进程的内存空间（eg：多个线程可以访问同一个共享变量）
- 线程更加轻量级，线程的上下切换的开销一般要比进程之间的上下文切换开销低

#### 并发与并行

##### 并发

单核CPU下，多个线程实际还是`串行执行`的。操作系统中有一个组件叫做`任务调度器`，可以将CPU的时间片（Windows下时间片最小为15毫秒）划分给不同的线程使用，由于CPU在不同之间的时间片切换是非常快的，对用户来说感知不到这个切换的过程，因此用户感觉线程是并行运行的。

总结：*微观串行，宏观并行*

<img src="C:\Users\lucas.zhao\AppData\Roaming\Typora\typora-user-images\image-20210615180503082.png" alt="image-20210615180503082" style="zoom:50%;" />

一般将线程轮流使用CPU的做法称之为并发（Concurrent）

##### 并行

多核CPU下，每个核心都可以调度运行线程，这个时候线程可以是并行的（Parallel）

##### 应用之异步调用

从方法的调用角度来讲，如果**需要等待**方法的结果才可以继续运行，这就是**同步**；如果**不需要等待**方法的结果就可以继续运行，这种就是**异步**

设计：多线程的使用可以让方法的执行变成异步的

##### 应用之提高效率

### 3. Java线程

#### 本章内容

1. 创建和运行线程
2. 查看线程
3. 线程API
4. 线程状态

#### 3.1 创建和运行线程

##### Thread

```java
@Slf4j(topic = "c.test")
public class TheadTest {
    public static void main(String[] args) {
        Thread thread = new Thread() {
            @Override
            public void run() {
                log.info("running...");
            }
        };
        thread.start();
        log.info("test");
    }
}
```

##### Runnable

```java
@Slf4j(topic = "RunnableTest")
public class RunnableTest {
    public static void main(String[] args) {
        Runnable runnable = new Runnable(){
            @Override
            public void run() {
                log.info("running...");
            }
        };
        Thread thread = new Thread(runnable, "t1");
        thread.start();

        lambda();
    }

    private static void lambda() {
        Thread thead1 = new Thread(() -> log.info("lambda test..."), "thead1");
        thead1.start();
    }
}
```

#### 3.2 查看和杀死进程

##### Windows

1. tasklist 查看进程

2. taskkill杀死进程

   ```shell
   tasklist | findstr java
   taskkill /F /PID 进程id
   ```

3. jps 查看java进程

##### Linux

1. ps -ef 查看所有进程信息
2. ps -fT -p <PID> 查看某个进程的所有线程信息
3. kill 杀死进程
4. top 实时查看进程信息
5. top -H -p <PID> 查看某个进程的所有线程信息

##### Java

1. jps 查看所有java进程

2. jstack <PID> 查看java进程的线程状态

3. jconsole 查看某个java进程的所有线程运行状态（图形化界面）

4. jconsole 远程监控配置

   ```shell
   java -Djava.rmi.server.hostname=`ip地址` -Dcom.sun.management.jmxremote -
   Dcom.sun.management.jmxremote.port=`连接端口` -Dcom.sun.management.jmxremote.ssl=是否安全连接 -
   Dcom.sun.management.jmxremote.authenticate=是否认证 java类
   ```

#### 3.3 线程运行原理

##### 栈和栈帧



##### 线程上下文切换





