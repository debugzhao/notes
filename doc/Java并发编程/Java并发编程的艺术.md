## 第1章 并发编程的挑战

### 1.1上下文切换

即使是单核处理器也支持多线程执行代码，CPU通过给每个线程分配CPU时间片来实现 这个机制。时间片是CPU分配给各个线程的时间，因为时间片非常短，所以CPU通过不停地切 换线程执行。

CPU通过时间片分配算法来循环执行任务，当前任务执行一个时间片后会切换到下一个任务。但是在切换前会保存上一个任务的状态，以便下次切换回这个任务时，可以再加载这 个任务的状态。所以任务从保存到再加载的过程就是一次上下文切换。

#### 1.1.1 多线程就一定快吗

当并发执行累加操作不超过百万次时，速度会比串行执行累加操作要慢，因为线程的创建和上下文切换都是有开销的。

#### 1.1.2 测试上下文切换次数和时长

- 使用Lmbench3 [1]可以测量上下文切换的时长
- 使用vmstat可以测量上下文切换的次数。

![](https://i.bmp.ovh/imgs/2022/05/23/4888316920f36051.png)

CS（Content Switch）表示上下文切换的次数，从上面的测试结果中我们可以看到，上下文 每1秒切换1000多次

#### 1.1.3 如何减少上下文切换

<font color="red">减少上下文切换的方法有无锁并发编程、CAS算法、使用最少线程和使用协程。</font>

- 无锁并发编程

  无锁并发编程。多线程竞争锁时，会引起上下文切换，所以多线程处理数据时，可以用一 些办法来避免使用锁，如将数据的ID按照Hash算法取模分段，不同的线程处理不同段的数据。

- CAS算法

  Java的Atomic包使用CAS算法来更新数据，而不需要加锁。

- 使用最少线程

  避免创建不需要的线程，比如任务很少，但是创建了很多线程来处理，这样会造成大量线程都处于等待状态。

- 使用协程

  在单线程里实现多任务的调度，并在单线程里维持多个任务间的切换。

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
                    try { 
                      Thread.sleep(2000);
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

### 2.1 volatile的应用❗❗

volatile是轻量级的 synchronized，它在多处理器开发中<font color="red">保证了共享变量的“可见性”</font>。可见性的意思是当一个线程 修改一个共享变量时，另外一个线程能读到这个修改的值。<font color="red">如果volatile变量修饰符使用恰当的话，它比synchronized的使用和执行成本更低，因为它不会引起线程上下文的切换和调度。</font>

#### volatile的定义与实现原理

如果一个字段被声明成volatile，Java线程内存模型确保所有线程看到这个变量的值是一致的。

1. 为了提高处理速度，处理器不直接和内存进行通信，而是先将系统内存的数据读到内部缓存后再进行操作，但操作完不知道何时会写到内存。
2. 如果对声明了volatile的 变量进行写操作，在多处理器下，为了保证各个处理器的缓存是一致的，就会实现缓存一 致性协议。
3. 每个处理器通过嗅探在总线上传播的数据来检查自己缓存的值是不是过期了，当处理器发现自己缓存行对应的内存地址被修改，就会将当前处理器的缓存行设置成无效状态。（总线嗅探机制）
4. 当处理器对这个数据进行修改操作的时候，会重新从系统内存中把数据读到处理器缓存里。

####  Volatile的两条实现原则

1. Lock前缀的指令会使处理器缓存的数据会写到内存中

   Lock前缀的指令一般会锁缓存而不会锁总线，因为锁总线的开销比较大

2. 一个处理器的缓存会写到内存会使其他的处理器缓存无效

#### volatile的使用优化

### 2.2 synchronized的实现原理与应用❗❗

本文详细介绍Java SE 1.6中为了减少获得锁和释放锁带来的性能消耗而引入的偏向锁和轻量级 锁，以及锁的存储结构和升级过程。

<font color="red">synchronized实现同步的三种方式：（Java中的每一个对象都都可以作为锁）</font>

1. 普通同步方法，锁的是当前实例对象

   ```java
   /**
    * 普通同步方法，锁的是当前实例对象
    */
   private synchronized void synchronizedMethod() {
   
   }
   ```

2. 静态同步方法，锁的是当前类的class对象

   ```java
   /**
    * 普通同步静态方法，锁的是当前类的class对象
    */
   private static synchronized void synchronizedStaticMethod() {
   
   }
   ```

3. 同步代码块，锁的是synchronized括号里配置的对象

   ```java
   /**
    * 同步代码块，锁的是括号配置的对象
    */
   private void synchronizedClass () {
       synchronized (this) {
       }
   }
   ```

<font color="red">当一个线程试图访问同步代码块时，它首先必须得到锁，退出或抛出异常时必须释放锁。</font>

Synchonized在JVM里的实现原理，JVM基于进入和退出Monitor对 象来实现方法同步和代码块同步，代码块同步是使用monitorenter 和monitorexit指令实现的。

 JVM基于进入和退出Monitor对象来实现方法同步和代码块同步，但是两者的实现方式不一样。代码块同步使用的是monitorenter指令和    monitroexit指令实现的，方法同步使用的是另一种方式实现的，但是方法同步也可以用monitorenter指令和monitroexit指令实现。

monitorenter指令在编译后插入到同步代码块的起始位置，monitorexit指令插入在方法结束处何异常处，JVM保证每一个monitorenter指令都有一个monitorexit指令与之配对。任何一个对象都有一个monitor与之相关联，当一个monitor被持有以后他将处于锁定状态。线程执行到monitorenter指令时会尝试获取对象所对应的monitor的所有权，即获取对应的锁。

#### 2.2.1 Java对象头

synchronized用的锁是存在Java对象头里的。Java对象头里的Mark Word里默认存储对象的HashCode、分代年龄和锁标记位。

![](https://s3.bmp.ovh/imgs/2022/05/23/a9a03c1b598e1274.png)

#### 2.2.2 锁的升级和对比❗❗

Java SE 1.6为了减少获得锁和释放锁带来的性能消耗，引入了“偏向锁”和“轻量级锁”。Java SE 1.6中，锁一共有4种状态，级别从低到高依次是：`无锁状态` < `偏向锁状态`  <  `轻量级锁状态`  < `重量级锁状态` 。<font color="red">这几个状态会随着竞争情况逐渐升级，锁可以升级但不能降级，目的是为了提高获得锁和释放锁的效率。</font>

##### 偏向锁

<font color="red">大多数情况下，锁不仅不存在多线程竞争，而且总是由同一线程多次获得，为了让线程获得锁的代价更低而引入了偏向锁。</font>当一个线程访问同步块并获取锁时，会在对象头和栈帧中的锁记录里存储锁偏向的线程ID，以后该线程在进入和退出同步块时不需要进行CAS操作来加锁和解锁，只需简单地测试一下对象头的Mark Word里是否存储着指向当前线程的偏向锁。

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

### 2.3 原子操作的实现原理❗❗

原子操作（atomic operation）意 为“不可被中断的一个或一系列操作”。

#### 术语定义

1. CAS
   CAS操作需要输入两个值，一个旧值（期望操作之前的值）和一个新值。在操作期间先比较旧值有没有发生变化，如果有发生变化则说明该值已经被其他线程修改过，则不进行交换；如果没有发生变化，则交换新的值。

#### 处理器如何实现原子操作

1. 使用总线锁保证原子性
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

   当对一个共享变量执行操作时，我们可以使用循 环CAS的方式来保证原子操作，但是对多个共享变量操作时，循环CAS就无法保证操作的原子性。 
   
   从JDK1.5开始，JDK提供了AtomicReference类来保证引用对象的原子性操作，可以将多个变量放到原子对象中来保证CAS操作。

##### 使用锁实现原子操作

<font color="red">锁机制保证了只有获得锁的线程才能够操作锁定的内存区域。</font>JVM内部实现了很多种锁 机制，有偏向锁、轻量级锁和互斥锁，

## 第3章 Java内存模型

### 3.1 <font color="red">Java内存模型基础❗❗</font>

#### 并发编程模型的两个关键问题

线程之间如何通信及线程之间如何同步。

1. 线程之间如何通信

   通信是指线程之间以何种机制来交换信息。在命令式编程中，线程之间的通信机制有两种：<font color="red">共享内存和消息传递。</font>

   在共享内存的并发模型里，线程之间共享程序的公共状态，通过<font color="red">写-读内存中的公共状态进行隐式通信</font>；在消息传递的并发模型里，线程之间没有公共状态，线程之间必须通过<font color="red">发送消息来显式进行通信。</font>

2. 线程之间如何同步

   同步是指程序中用于控制不同线程间操作发生相对顺序的机制。

<font color="red">Java的并发采用的是共享内存模型，Java线程之间的通信总是隐式进行，整个通信过程对 程序员完全透明。</font>

#### Java内存模型的抽象结构

Java线程之间的通信由Java内存模型（本文简称为JMM）控制，JMM决定一个线程对共享 变量的写入何时对另一个线程可见。

<font color="red">线程之间的共享变量存储在主内存（Main Memory）中，每个线程都有一个私有的本地 内存（Local Memory），本地内存中存储了该线程以读/写共享变量的副本。</font>

<img src="https://i.bmp.ovh/imgs/2022/05/24/41b43dcbe2ce455d.png" style="zoom:67%;" />

线程之间的通信步骤：

1. 线程A把本地内存A中更新过的共享变量刷新到主内存中去。
2. 线程B到主内存中去读取线程A之前已更新过的共享变量。

<img src="https://i.bmp.ovh/imgs/2022/05/24/2027261227b4d783.png" style="zoom:67%;" />

<font color="red">JMM通过控制主内存与每个线程的本地内存之间的交互，来为Java程序员提供 内存可见性保证。</font>

#### 从源代码到指令序列的重排序❗❗

在执行程序时，为了提高性能，编译器和处理器常常会对指令做重排序。

![](https://i.bmp.ovh/imgs/2022/05/24/9573a9f685eed1bd.png)

<font color="red">这些重排序可能会导致多线程程序出现内存可见性问题</font>，JMM属于语言级的内存模型，它确保在不同的编译器和不同的处理器平台之上，<font color="red">通过禁止特定类型的编译器重排序和处理器重排序，为程序员提供一致的内存可见性保证。</font>

#### 并发编程模型的分类

#### <font color="red">happens-before简介</font>

在JMM中，如果一 个操作执行的结果需要对另一个操作可见，那么这两个操作之间必须要存在happens-before关 系

### 3.2 重排序❗❗

重排序是指编译器和处理器为了优化程序性能而对指令序列进行重新排序的一种手段

### 3.3 顺序一致性

### 3.4 volatile的内存语义

### 3.5 锁的内存语义

### 3.6 finale域的内存语义

### 3.7 happens-before原则❗❗

### 3.8 双重检查锁与延迟初始化

### 3.9 Java内存模型综述

## 第4章 Java并发编程基础

### 4.1 线程简介

#### 什么是线程

#### 为什么要使用多线程

1. 更多的处理器核心，提高资源的利用率
2. 更快的响应时间
3. 更好的编程模型

#### 线程的优先级

<font color="red">线程优先级就是决定线程需 要多或者少分配一些处理器资源的线程属性，</font>优先级高的线程分 配时间片的数量要多于优先级低的线程。

 **线程优先级不能作为程序正确性的依赖，因为操作系统可以完全不用理会Java 线程对于优先级的设定**

#### 线程的状态	

![](https://i.bmp.ovh/imgs/2022/05/24/004de9b95b994c7f.png)

1. 初始状态

   线程被创建，但是还没有调用start方法

2. 运行状态

   线程创建之后，调用start()方法开始运行

3. 等待状态

   当线程执行wait()方法之 后，线程进入等待状态，进入等待状态的线程需要依靠其他线程的通知才能够返回到运行状态

4. 阻塞状态

   当线程调用同步方法时，在没有获取到锁的情况下，线程将会进入到阻塞状态

5. 终止状态

   线程在执行Runnable的run()方法之后将会进入到终止状态

#### Daemon线程

Daemon线程是一种支持型线程，因为它主要被用作程序中后台调度以及支持性工作。

*在构建Daemon线程时，不能依靠finally块中的内容来确保执行关闭或清理资源的逻辑。*

### 4.2 启动和终止线程

#### 理解中断

中断可以理解为线程的一个标识位属性，它表示一个运行中的线程是否被其他线程进行 了中断操作

#### 过期的suspend()、resume()和stop()

suspend()、resume()和stop()方法完成了线程的暂停、恢复和终 止工作，而且非常“人性化”。<font color="red">但是这些API是过期的，也就是不建议使用的。</font>

不建议使用的原因主要有：以suspend()方法为例，在调用后，线程不会释放已经占有的资 源（比如锁），而是占有着资源进入睡眠状态，这样容易引发死锁问题。 

#### <font color="red">优雅地终止线程❗❗</font>

的中断状态是线程的一个标识位，而中断操作是一种简便的线程间交互 方式，而这种交互方式最适合用来取消或停止任务。除了中断以外，还可以利用一个boolean变 量来控制是否需要停止任务并终止该线程。

```java
public class Shutdown {
    public static void main(String[] args) throws Exception {
        Runner one = new Runner();
        Thread countThread = new Thread(one, "CountThread1");
        countThread.start();
        // 睡眠1秒，main线程对CountThread进行中断，使CountThread能够感知中断而结束
        TimeUnit.SECONDS.sleep(1);
        countThread.interrupt();

        Runner two = new Runner();
        countThread = new Thread(two, "CountThread2");
        countThread.start();
        // 睡眠1秒，main线程对Runner two进行取消，使CountThread能够感知on为false而结束
        TimeUnit.SECONDS.sleep(1);
        two.cancel();
    }


    private static class Runner implements Runnable {
        private long i;
        private volatile boolean on = true;
        @Override
        public void run() {
            while (on && !Thread.currentThread().isInterrupted()){
                i++;
            }
            System.out.println(Thread.currentThread().getName() + " Count i = " + i);
        }
        public void cancel() {
            on = false;
        }
    }
}
```

main线程通过中断操作和cancel()方法均可使CountThread得以终止。 这种通过标识位或者中断操作的方式能够使<font color="red">线程在终止时有机会去清理资源</font>，而不是武断地将线程停止，因此<font color="red">这种终止线程的做法显得更加安全和优雅</font>

### 4.3 线程间通信

#### volatile和synchronized关键字

Java支持多个线程同时访问一个对象或者对象的成员变量，由于每个线程可以拥有这个变量的拷贝，所以程序在执行过程中，一个线程看到的变量并不一定是最新的。

<font color="red">关键字volatile可以保证所有线程对共享变量的访问的可见性。就是告知线程对共享变量的访问需要从主存中获取，而对共享变量的改变必须同步刷新回主存。</font>

<font color="red">关键字synchronized可以用来修饰同步方法或者同步代码块，它可以用来确保多个线程在同一时刻只能有一个线程执行同步方法或者同步代码块，保证了线程对同步方法/同步代码块的可见性和排他性</font>

#### 等待/通知机制❗❗

```java
while (value != desire) {
	Thread.sleep(1000);
}
doSomething();	
```

上述这段循环检查代码弊端：

1. 难以确保及时性

   在睡眠时，基本不消耗处理器资源，但是如果睡得过久，就不能及时 发现条件已经变化，也就是及时性难以保证

2. 难以减低开销

   如果降低睡眠的时间，比如休眠1毫秒，这样消费者能更加迅速地发现 条件变化，但是却可能消耗更多的处理器资源，造成了无端的浪费

以上两个问题，看似矛盾难以调和，<font color="red">但是Java通过内置的等待/通知机制能够很好地解决 这个矛盾并实现所需的功能。</font>

| 方法名称        | 描述                                                         |
| --------------- | ------------------------------------------------------------ |
| notify()        | 通知一个在对象上等待的线程，并使其从wait()方法返回<br />而返回的前提是线程获取了到对象的锁 |
| notifyAll()     | 通知所有在等待该对象上的线程                                 |
| wait()          | 调用该方法的线程会进入WAITING状态，只有被其他线程通知或者中断才能返回<br />调用wait方法后，会释放对象的锁 |
| wait(long)      | 超时等待一段时间，如果没有通知就超时返回                     |
| wait(long, int) | 更加细粒度的超时等待，可以达到纳秒级别                       |

```java
public class WaitNotify {
    static boolean flag = true;
    static Object lock = new Object();
    public static void main(String[] args) throws Exception {
        Thread waitThread = new Thread(new Wait(), "WaitThread");
        waitThread.start();
        TimeUnit.SECONDS.sleep(1);
        Thread notifyThread = new Thread(new Notify(), "NotifyThread");
        notifyThread.start();
    }
    static class Wait implements Runnable {
        @Override
        public void run() {
            // 加锁，拥有lock的Monitor
            synchronized (lock) {
                // 当条件不满足时，继续wait，同时释放了lock的锁
                while (flag) {
                    try {
                        System.out.println(Thread.currentThread() + " flag is true. wait @ " + new SimpleDateFormat("HH:mm:ss").format(new Date()));
                        lock.wait();
                    } catch (InterruptedException e) {
                    }
                }
                // 条件满足时，完成工作
                System.out.println(Thread.currentThread() + " flag is false. running @ " + new SimpleDateFormat("HH:mm:ss").format(new Date()));
            }
        }
    }
    static class Notify implements Runnable {
        @SneakyThrows
        @Override
        public void run() {
            // 加锁，拥有lock的Monitor
            synchronized (lock) {
                // 获取lock的锁，然后进行通知，通知时不会释放lock的锁，
                // 直到当前线程释放了lock后，WaitThread才能从wait方法中返回
                System.out.println(Thread.currentThread() + " hold lock. notify @ " +  new SimpleDateFormat("HH:mm:ss").format(new Date()));
                lock.notifyAll();
                flag = false;
                TimeUnit.SECONDS.sleep(5);
            }
            // 再次加锁
            synchronized (lock) {
                System.out.println(Thread.currentThread() + " hold lock again. sleep  @ " + new SimpleDateFormat("HH:mm:ss").format(new Date()));
                TimeUnit.SECONDS.sleep(5);
            }
        }
    }
}
```

![](https://s3.bmp.ovh/imgs/2022/05/26/3eaac49c9b96a88d.png)    

在图4-3中，WaitThread首先获取了对象的锁，然后调用对象的wait()方法，从而放弃了锁 并进入了对象的等待队列WaitQueue中，进入等待状态。由于WaitThread释放了对象的锁， NotifyThread随后获取了对象的锁，并调用对象的notify()方法，将WaitThread从WaitQueue移到 SynchronizedQueue中，此时WaitThread的状态变为阻塞状态。NotifyThread释放了锁之后， WaitThread再次获取到锁并从wait()方法返回继续执行

#### 等待/通知机制的经典范式

从4.3.2节中的WaitNotify示例中可以提炼出等待/通知的经典范式，该范式分为两部分，分 别针对等待方（消费者）和通知方（生产者）。

1. 等待方遵循如下原则

   1. 获取对象的锁
   2. 条件不满足则继续等待，调用对象的wait()方法，被通知后任然要检查条件
   3. 条件满足则执行对应的逻辑

   ```java
   synchronized(obj) {
       while(条件不满足) {
           obj.lock();
       }
       条件满足则处理对应的逻辑
   }
   ```

2. 通知方遵循如下原则

   1. 获取对象的锁
   2. 改变条件
   3. 通知所有在该对象上等待的线程

   ```java
   synchronized(obj) {
       改变条件
       obj.notifyAll()
   }
   ```

#### 管道输入/输出流

管道输入/输出流和普通的文件输入/输出流或者网络输入/输出流不同之处在于，它主要 用于线程之间的数据传输，而传输的媒介为内存。

*对于Piped类型的流，必须先要进行绑定，也就是调用connect()方法，如果没有将输入/输 出流绑定起来，对于该流的访问将会抛出异常。*

#### Thread.join()的使用❗❗

如果一个线程A执行了thread.join()语句，其含义是：当前线程A等待thread线程终止之后才 从thread.join()返回。

```java
public class Join {
    public static void main(String[] args) throws Exception {
        Thread previous = Thread.currentThread();
        for (int i = 0; i < 10; i++) {
            // 每个线程拥有前一个线程的引用，需要等待前一个线程终止，才能从等待中返回
            Thread thread = new Thread(new Domino(previous), String.valueOf(i));
            thread.start();
            previous = thread;
        }
        TimeUnit.SECONDS.sleep(5);
        System.out.println(Thread.currentThread().getName() + " terminate.");
    }
    static class Domino implements Runnable {
        private Thread thread;
        public Domino(Thread thread) {
            this.thread = thread;
        }
        @Override
        public void run() {
            try {
                thread.join();
            } catch (InterruptedException e) {
            }
            System.out.println(Thread.currentThread().getName() + " terminate.");
        }
    }
}
```

```java
main terminate.
0 terminate.
1 terminate.
2 terminate.
3 terminate.
4 terminate.
5 terminate.
6 terminate.
7 terminate.
8 terminate.
9 terminate.
```

从上述输出可以看到，每个线程终止的前提是前驱线程的终止，每个线程等待前驱线程 终止后，才从join()方法返回，这里涉及了等待/通知机制（等待前驱线程结束，接收前驱线程结 束通知）。

#### ThreadLocal的使用❗❗

ThreadLocal，即线程变量，是一个以ThreadLocal对象为键、任意对象为值的存储结构。这 个结构被附带在线程上，也就是说一个线程可以根据一个ThreadLocal对象查询到绑定在这个 线程上的一个值

可以通过set(T)方法来设置一个值，在当前线程下再通过get()方法获取到原先设置的值

```java
public class Profiler {
    // 第一次get()方法调用时会进行初始化（如果set方法没有调用），每个线程会调用一次
    private static final ThreadLocal<Long> TIME_THREADLOCAL = new ThreadLocal<Long>() {
        @Override
        protected Long initialValue() {
            return System.currentTimeMillis();
        }
    };

    public static final void begin() {
        TIME_THREADLOCAL.set(System.currentTimeMillis());
    }

    public static final long end() {
        return System.currentTimeMillis() - TIME_THREADLOCAL.get();
    }

    public static void main(String[] args) throws Exception {
        Profiler.begin();
        TimeUnit.SECONDS.sleep(1);
        System.out.println("Cost: " + Profiler.end() + " mills");
    }
}
```

### 4.4 线程应用实例

#### 等待超时模式

#### 一个简单的数据库连接池示例

#### 线程池技术及其示例

可以看到，<font color="red">线程池的本质就是使用了一个线程安全的工作队列连接工作者线程和客户端线程，</font>客户端线程将任务放入工作队列后便返回，而工作者线程则不断地从工作队列上取出 工作并执行。当工作队列为空时，所有的工作者线程均等待在工作队列上，当有客户端提交了 一个任务之后会通知任意一个工作者线程，随着大量的任务被提交，更多的工作者线程会被唤醒。

#### 一个基于线程池技术的简单web服务器

## 第5章 Java中的锁

### 5.1 Lock接口

<font color="red">锁是用来控制多个线程访问共享资源的方式</font>，一般来说，一个锁能够防止多个线程同时 访问共享资源（但是有些锁可以允许多个线程并发的访问共享资源，比如读写锁、可重入锁）。

在Java SE 5之后，并发包中新增 了Lock接口（以及相关实现类）用来实现锁功能，它提供了与synchronized关键字类似的同步功能，<font color="red">只是在使用时需要显式地获取和释放锁。</font>虽然它缺少了（通过synchronized块或者方法所提 供的）隐式获取释放锁的便捷性，<font color="red">但是却拥有了锁获取与释放的可操作性、可中断的获取锁以及超时获取锁等多种synchronized关键字所不具备的同步特性。</font>

Lock的使用方式

```java
// 在finally块中释放锁，目的是保证在获取到锁之后，最终能够被释放。
Lock lock = new ReentrantLock();
lock.lock();
try {
    
} finally {
	lock.unlock();	
}
```

*不要将获取锁的过程写在try块中，因为如果在获取锁（自定义锁的实现）时发生了异常， 异常抛出的同时，也会导致锁无故释放。*

Lock接口提供的synchronized关键字不具备的主要特性

| 特性               | 描述                                                         |
| ------------------ | ------------------------------------------------------------ |
| 尝试非阻塞地获取锁 | 当前线程尝试获取锁，如果这一时刻锁没有被其他线程获取到，则成功获取并持有锁 |
| 能被中断地获取锁   | 获取到锁的线程可以响应中断，当被中断时中断异常被抛出，并且释放锁 |
| 超时获取锁         | 在指定时间之前获取锁，如果到了时间仍无法获取则直接返回       |

### 5.2 队列同步器

#### 队列同步器的接口与实例

#### 队列同步器的实现分析

### 5.3 重入锁

重入锁ReentrantLock，顾名思义，就是支持重进入的锁，<font color="red">它表示该锁能够支持一个线程对资源的重复加锁</font>。除此之外，该锁的还支持获取锁时的公平和非公平性选择。

ReentrantLock在调用lock()方法时，已经获取到锁的线程，能够再次调用lock()方法获取锁而不被阻塞。

事实上，<font color="red">公平的锁机制往往没有非公平的效率高</font>，但是，并不是任何场景都是以TPS作为 唯一的指标，公平锁能够减少“饥饿”发生的概率，等待越久的请求越是能够得到优先满足。

### 5.4 读写锁

#### 读写锁的接口与实例

#### 读写锁的实现分析

### 5.5 LockSupport工具

### 5.6 Condition接口

#### Condition接口接口与实例

#### Condition接口的实现分析







<font color="red"></font>
