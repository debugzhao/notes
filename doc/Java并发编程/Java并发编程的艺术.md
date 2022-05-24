## 第1章 并发编程的挑战

### 1.1上下文切换

即使是单核处理器也支持多线程执行代码，CPU通过给每个线程分配CPU时间片来实现 这个机制。时间片是CPU分配给各个线程的时间，因为时间片非常短，所以CPU通过不停地切 换线程执行。

CPU通过时间片分配算法来循环执行任务，当前任务执行一个时间片后会切换到下一个 任务。但是，在切换前会保存上一个任务的状态，以便下次切换回这个任务时，可以再加载这 个任务的状态。所以任务从保存到再加载的过程就是一次上下文切换。

#### 1.1.1 多线程就一定快吗

当并发执行累加操作不超过百万次时，速度会比串行执行累加操作要慢，因为线程的创建和上下文切换都是有开销的。

#### 1.1.2 测试上下文切换次数和时长

- 使用Lmbench3 [1]可以测量上下文切换的时长
- 使用vmstat可以测量上下文切换的次数。

![](https://i.bmp.ovh/imgs/2022/05/23/4888316920f36051.png)

CS（Content Switch）表示上下文切换的次数，从上面的测试结果中我们可以看到，上下文 每1秒切换1000多次

#### 1.1.3 如何减少上下文切换

减少上下文切换的方法有无锁并发编程、CAS算法、使用最少线程和使用协程。

- 无锁并发编程

  无锁并发编程。多线程竞争锁时，会引起上下文切换，所以多线程处理数据时，可以用一 些办法来避免使用锁，如将数据的ID按照Hash算法取模分段，不同的线程处理不同段的数据。

- CAS算法

  Java的Atomic包使用CAS算法来更新数据，而不需要加锁。

- 使用最少线程

  避免创建不需要的线程，比如任务很少，但是创建了很多线程来处理，这样会造成大量线程都处于等待状态。

- 使用协程

  在单线程里实现多任务的调度，并在单线程里维持多个任务间的切换

#### 1.1.4 减少上下文切换实战

### 1.2 死锁

锁是个非常有用的工具，运用场景非常多，因为它使用起来非常简单，而且易于理解。但 同时它也会带来一些困扰，那就是可能会引起死锁，一旦产生死锁，就会造成系统功能不可用。

```java
public class DeadLockDemo {
    private static String A = "A";
    private static String B = "B";
    public static void main(String[] args) {
        new DeadLockDemo().deadLock();
    }
    private void deadLock() {
        Thread t1 = new Thread(new Runnable() {
            @Override
              public void run() {
                synchronized (A) {
                    try { Thread.sleep(2000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    synchronized (B) {
                        System.out.println("1");
                    }
                }
            }
        });
        Thread t2 = new Thread(new Runnable() {
            @Override
            public void run() {
                synchronized (B) {
                    synchronized (A) {
                        System.out.println("2");
                    }
                }
            }
        });
        t1.start();
        t2.start();
    }
}
```

这段代码会引起死锁，<font color="red">使线程t1和线程t2互相等待对方释放锁。</font>在复杂的业务场景中可能还会遇到这种情况：<font color="red">比如t1拿到锁之后，因为一些异常情况没有释放锁 （死循环）。</font>

一旦出现死锁，业务是可感知的，因为不能继续提供服务了，那么只能通过dump线程查看 到底是哪个线程出现了问题。

<font color="red">避免死锁的几种方法：</font>

- 避免一个线程同时获取多个锁
- 避免一个线程在锁内同时占用多个资源，尽量保证一个锁只占用一个资源
- 尝试使用<font color="red">定时锁(tryLock)</font>来代替内部锁机制
- 对于数据库锁，加锁和解锁必须在同一个数据库连接中，否则会出现解锁失败的情况

### 1.3 资源限制的挑战

- 硬件资源限制
  - 网卡带宽限制
  - 磁盘读写速度限制
  - CPU处理速度限制
- 软件资源限制
  -  数据库的连接数限制
  - socket连接数限制

## 第2章 Java并发编程的底层实现原理

Java代码的底层执行过程：<font color="red">Java代码在编译后会变成Java字节码，字节码经过类加载器加载到JVM中，JVM执行字节码，最终转成成汇编指令在CPU上执行。Java中使用的并发机制依赖于JVM的实现和CPU的指令</font>

### 2.1 volatile的应用

volatile是轻量级的 synchronized，它在多处理器开发中<font color="red">保证了共享变量的“可见性”</font>。可见性的意思是当一个线程 修改一个共享变量时，另外一个线程能读到这个修改的值。
<font color="red">如果volatile变量修饰符使用恰当 的话，它比synchronized的使用和执行成本更低，因为它不会引起线程上下文的切换和调度。</font>

#### volatile的定义与实现原理

如果一个字段被声明成volatile，Java线程内存模型确保所有线程看到这个变量的值是一致的。

1. 为了提高处理速度，处理器不直接和内存进行通信，而是先将系统内存的数据读到内部缓存后再进行操作，但操作完不知道何时会写到内存。
2. 如果对声明了volatile的 变量进行写操作，在多处理器下，为了保证各个处理器的缓存是一致的，就会实现缓存一 致性协议。
3. 每个处理器通过嗅探在总线上传播的数据来检查自己缓存的值是不是过期了，当处理器发现自己缓存行对应的内存地址被修改，就会将当前处理器的缓存行设置成无效状态。
4. 当处理器对这个数据进行修改操作的时候，会重新从系统内存中把数据读到处理器缓存里。

#### volatile的使用优化

### 2.2 synchronized的实现原理与应用

本文详细介绍Java SE 1.6中为了减少获得锁和释放锁带来的性能消耗而引入的偏向锁和轻量级 锁，以及锁的存储结构和升级过程。

<font color="red">synchronized实现同步的三种方式：</font>

1. 普通同步方法，锁的是当前实例对象
2. 静态同步方法，锁的是当前类的class对象
3. 同步代码块，锁的是synchronized括号里配置的对象

<font color="red">当一个线程试图访问同步代码块时，它首先必须得到锁，退出或抛出异常时必须释放锁。</font>

Synchonized在JVM里的实现原理，JVM基于进入和退出Monitor对 象来实现方法同步和代码块同步，代码块同步是使用monitorenter 和monitorexit指令实现的。

#### 2.2.1 Java对象头

synchronized用的锁是存在Java对象头里的。Java对象头里的Mark Word里默认存储对象的HashCode、分代年龄和锁标记位。

![](https://s3.bmp.ovh/imgs/2022/05/23/a9a03c1b598e1274.png)

#### 2.2.2 锁的升级和对比

Java SE 1.6为了减少获得锁和释放锁带来的性能消耗，引入了“偏向锁”和“轻量级锁”。Java SE 1.6中，锁一共有4种状态，级别从低到高依次是：`无锁状态` < `偏向锁状态`  <  `轻量级锁状态`  < `重量级锁状态` 。<font color="red">这几个状态会随着竞争情况逐渐升级，锁可以升级但不能降级，目的是为了提高获得锁和释放锁的效率。</font>

##### 偏向锁

<font color="red">大多数情况下，锁不仅不存在多线程竞争，而且总是由同一线程多次获得，为了让线程获得锁的代价更低而引入了偏向锁。</font>当一个线程访问同步块并获取锁时，会在对象头和栈帧中的锁记录里存储锁偏向的线程ID，以后该线程在进入和退出同步块时不需要进行CAS操作来加锁和解锁，只需简单地测试一下对象头的Mark Word里是否 存储着指向当前线程的偏向锁。

1. 偏向锁的撤销

   偏向锁使用了一种等到竞争出现才释放锁的机制，所以当其他线程尝试竞争偏向锁时， 持有偏向锁的线程才会释放锁

2. 关闭偏向锁

   ```shell
   -XX:-UseBiasedLocking=false
   ```

   关闭程序默认会进入轻量级锁状态

##### 轻量级锁

1. 轻量级锁加锁过程

   1. 线程在执行同步块之前，JVM会先在当前线程的栈桢中创建用于存储锁记录的空间，并将对象头中的Mark Word复制到锁记录中。
   2. 然后线程尝试使用 CAS将对象头中的Mark Word替换为指向锁记录的指针
   3. 如果成功，当前线程获得锁，如果失败，表示其他线程竞争锁，当前线程便尝试使用<font color="red">自旋来获取锁</font>。

2. 轻量级锁解锁过程

   轻量级解锁时，会使用原子的CAS操作将Displaced Mark Word替换回到对象头，如果成功，则表示没有竞争发生。如果失败，表示当前锁存在竞争，锁就会膨胀成重量级锁。

##### 锁的优缺点对比

| 锁类型   | 优点                                                         | 缺点                                             | 适用场景                               |
| -------- | ------------------------------------------------------------ | ------------------------------------------------ | -------------------------------------- |
| 偏向锁   | 加锁和解锁不需要额外的消耗<br />和执行非同步方法时间仅存在纳秒级别的差距 | 如果线程之间存在锁竞争，会带来额外的锁撤销的消耗 | 适用于只有一个线程访问同步代码块的场景 |
| 轻量级锁 | 竞争的线程不会阻塞，提高了程序的相应速度                     | 如果始终得不到锁的线程，会自旋消耗CPU            | 追求相应时间<br />同步块执行速度非常快 |
| 重量级锁 | 线程竞争不会使用自旋，不会消耗CPU                            | 线程阻塞，相应时间缓慢                           | 追求吞吐量                             |

> 什么是自旋？？

### 2.3 原子操作的实现原理

原子操作（atomic operation）意 为“不可被中断的一个或一系列操作”。

#### 术语定义

1. CAS
   CAS操作需要输入两个值，一个旧值（期望操作之前的值）和一个新值。在操作期间先比较旧值有没有发生变化，如果有发生变化则说明该值已经被其他线程修改过，则不进行交换；如果没有发生变化，则交换新的值。

#### 处理器如何实现原子操作

1. 使用总线保证原子性
2. 使用缓存锁保证原子性

#### Java如何实现原子操作

<font color="red">Java中可以通过锁和循环CAS的方式实现原子性。</font>

##### 使用循环CAS实现原则操作

<font color="red">自旋CAS实现的基本 思路就是循环进行CAS操作直到成功为止。</font>

```java
public class Counter {
    private AtomicInteger atomicI = new AtomicInteger(0);
    private int           i       = 0;

    public static void main(String[] args) {
        final Counter cas = new Counter();
        List<Thread> ts = new ArrayList<Thread>(600);
        long start = System.currentTimeMillis();
        for (int j = 0; j < 100; j++) {
            Thread t = new Thread(new Runnable() {
                @Override
                public void run() {
                    for (int i = 0; i < 10000; i++) {
                        cas.count();
                        cas.safeCount();
                    }
                }
            });
            ts.add(t);
        }
        for (Thread t : ts) {
            t.start();

        }

        // 等待所有的子线程执行完成
        for (Thread t : ts) {
            try {
                t.join();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

        }
        System.out.println(cas.i);
        System.out.println(cas.atomicI.get());
        System.out.println(System.currentTimeMillis() - start);
    }

    /**
     * 通过CAS操作实现的线程安全的方法
     */
    private void safeCount() {
        for (;;) {
            int i = atomicI.get();
            boolean suc = atomicI.compareAndSet(i, ++i);
            if (suc) {
                break;
            }
        }
    }

    /**
     * 非线程安全方法
     */
    private void count() {
        i++;
    }
}
```

##### CAS带来的三大问题

1. ABA问题

   因为CAS需要在操作值的时候，检查值有没有发生变化，如果没有发生变化 则更新，但是如果一个值原来是A，变成了B，又变成了A，那么使用CAS进行检查时会发现它 的值没有发生变化，但是实际上却变化了。

   <font color="red">ABA问题的解决思路就是使用版本号。在变量前面 追加上版本号，每次变量更新的时候把版本号加1，那么A→B→A就会变成1A→2B→3A。</font>

2. 循环时间长开销大

   自旋CAS如果长时间不成功，会给CPU带来非常大的执行开销。

3. 只能保证一个共享变量的原则操作

   当对一个共享变量执行操作时，我们可以使用循 环CAS的方式来保证原子操作，但是对多个共享变量操作时，循环CAS就无法保证操作的原子性

##### 使用锁实现原子操作

<font color="red">锁机制保证了只有获得锁的线程才能够操作锁定的内存区域。</font>JVM内部实现了很多种锁 机制，有偏向锁、轻量级锁和互斥锁，

## 第3章 Java内存模型

### 3.1 Java内存模型基础

#### 并发编程模型的两个关键问题

线程之间如何通信及线程之间如何同步。

1. 线程之间如何通信

   通信是指线程之间以何种机制来交换信息。在命令式编程中，线程之间的通信机制有两种：<font color="red">共享内存和消息传递。</font>

   在共享内存的并发模型里，线程之间共享程序的公共状态，通过<font color="red">写-读内存中的公共状态进行隐式通信</font>；在消息传递的并发模型里，线程之间没有公共状态，线程之间必须通过<font color="red">发送消息来显式进行通信。</font>

2. 线程之间如何同步

   同步是指程序中用于控制不同线程间操作发生相对顺序的机制。

<font color="red">Java的并发采用的是共享内存模型，Java线程之间的通信总是隐式进行，整个通信过程对 程序员完全透明。</font>

1. 

#### Java内存模型的抽象结构

Java线程之间的通信由Java内存模型（本文简称为JMM）控制，JMM决定一个线程对共享 变量的写入何时对另一个线程可见。

<font color="red">线程之间的共享变量存储在主内存（Main Memory）中，每个线程都有一个私有的本地 内存（Local Memory），本地内存中存储了该线程以读/写共享变量的副本。</font>

<img src="https://i.bmp.ovh/imgs/2022/05/24/41b43dcbe2ce455d.png" style="zoom:67%;" />

线程之间的通信步骤：

1. 线程A把本地内存A中更新过的共享变量刷新到主内存中去。
2. 线程B到主内存中去读取线程A之前已更新过的共享变量。

<img src="https://i.bmp.ovh/imgs/2022/05/24/2027261227b4d783.png" style="zoom:67%;" />

#### 从源代码到指令序列的重排序

在执行程序时，为了提高性能，编译器和处理器常常会对指令做重排序。

![](https://i.bmp.ovh/imgs/2022/05/24/9573a9f685eed1bd.png)

<font color="red">这些重排序可能会导致多线程程序出现内存可见性问题</font>，JMM属于语言级的内存模型，它确保在不同的编译器和不同的处理器平台之上，<font color="red">通过禁止特定类型的编译器重排序和处理器重排序，为程序员提供一致的内存可见性保证。</font>

#### 并发编程模型的分类

#### <font color="red">happens-before简介</font>

在JMM中，如果一 个操作执行的结果需要对另一个操作可见，那么这两个操作之间必须要存在happens-before关 系

### 3.2 重排序

### 3.3 顺序一致性

### 3.4 volatile的内存语义

### 3.5 锁的内存语义

### 3.6 finale域的内存语义

### 3.7 happens-before原则

### 3.8 双重检查锁与延迟初始化

### 3.9 Java内存模型综述

### 3.10 本章小结









<font color="red"></font>
