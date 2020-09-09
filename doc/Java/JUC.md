### 1.进程和线程

#### 1.1进程和线程

进程：一个活动的程序叫做进程

线程：CPU操作程序的最小执行单元（一个进程至少包括一个线程）

*Java默认开启两个线程：main线程和GC线程*

对于Java而言，开启线程有三种方式，Thread、Runable、Callable。Callable和前两者相比可以有返回值，在企业开发中更加常用。

Java真的可以直接开启线程吗？答案是：不可以

```java
public synchronized void start() {

    if (threadStatus != 0)
    	throw new IllegalThreadStateException();
    // 添加至进程组
    group.add(this);

    boolean started = false;
    try {
        start0();
        started = true;
    } finally {
        try {
            if (!started) {
            group.threadStartFailed(this);
            }
        } catch (Throwable ignore) {
        }
        }
    }
// 启动native本地方法开启线程，本质是调用c++与CPU交互，Java不能直接与硬件交互
private native void start0();
```

#### 1.2并发和并行

并发：单核CPU，模拟出多条线程，CPU快速轮流执行

并行：多核CPU，开启多条线程，CPU同时执行

```java
public static void main(String[] args) {
    // 获取CPU核数（CPU密集型、IO密集型）
    System.out.println(Runtime.getRuntime().availableProcessors());
}
```

> 线程状态

- 新建
- 就绪
- 运行
- 阻塞
- 死亡

> wait/sleep区别

1. 自不同的类

   new Object.wait()    new Thread.sleep()

2. 关于锁的释放

   wait()会释放锁；sleep()方法，是死死地抱着锁睡觉的，不会释放锁

3. 使用范围不同

   wait()必须在同步代码块中执行，sleep()方法可以在任何地方执行

4. 是否需要捕获异常

   sleep()方法需要捕获异常、wait()方法不需要捕获异常

### 2.Lock锁

#### 2.1 synchronized

```java
public class Test {
    public static void main(String[] args) {
        Ticket ticket = new Ticket();
        new Thread(()->{
            for (int i = 0; i < 40; i++) {
                ticket.sale();
            }
        },"A").start();
        new Thread(()->{
            for (int i = 0; i < 40; i++) {
                ticket.sale();
            }
        },"B").start();
        new Thread(()->{
            for (int i = 0; i < 40; i++) {
                ticket.sale();
            }
        },"C").start();

    }
}

class Ticket{
    private int totalNum = 30;
    private int saledNum = 0;
    
    synchronized void sale(){
        saledNum ++;
        if(totalNum - saledNum >= 0){
            System.out.println("线程" + Thread.currentThread().getName() + "卖了" +      			 saledNum + "张票，剩余" + (totalNum - saledNum) +"张票");
        }
    }
}

```

#### 2.2 Lock锁

`Lock接口的所有实现类`

```java
// 可重入锁(普通锁)
ReentrantLock
// 读锁
ReentrantReadWriteLock.ReadLock
// 写锁
ReentrantReadWriteLock.WriteLock
```

`Lock锁的用法`

```java
// 以可重入锁为例
Lock lock = new ReentrantLock();
// 加锁
lock.lock();
try{
    // 业务代码
}catch(){
    
}finally{
    // 解锁
    lock.unlock();
}
```

##### 2.2.1 ReentrantLock

`ReentrantLock构造方法`

```
// 无参构造方法(默认非公平锁)
public ReentrantLock() {
	sync = new NonfairSync();
}

public ReentrantLock(boolean fair) {
	sync = fair ? new FairSync() : new NonfairSync();
}
```

`ReentrantLock锁分类`

- 非公平锁

  不遵循先来后到的原则，我可以插队（ReentrantLock锁默认为非公平锁）

- 公平锁

  遵循先来后到的原则

`ReentrantLock案例`

```java
@SuppressWarnings("all")
public class ReentrantLockDemo {
    public static void main(String[] args) {
        Ticket2 ticket = new Ticket2();
        new Thread(()->{
            for (int i = 0; i < 40; i++) {
                ticket.sale();
            }
        },"A").start();
        new Thread(()->{
            for (int i = 0; i < 40; i++) {
                ticket.sale();
            }
        },"B").start();
        new Thread(()->{
            for (int i = 0; i < 40; i++) {
                ticket.sale();
            }
        },"C").start();

    }
}

class Ticket2{
    private int totalNum = 30;
    private int saledNum = 0;

    Lock lock = new ReentrantLock();

    void sale(){
        lock.lock();

        try {
            saledNum ++;
            if(totalNum - saledNum >= 0){
                System.out.println("线程" + Thread.currentThread().getName() + "卖了" + 				saledNum + "张票，剩余" + (totalNum - saledNum) +"张票");
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
}
```



#### 2.3 synchronized和Lock锁区别

1. synchronized是Java内置关键字；Lock是个接口
2. synchronized无法判断获取锁的状态；Lock可以判断是否获取到了锁
3. synchronized可以自动释放锁；Lock必须要手动释放锁，否则会造成**死锁**的情况
4. 使用synchronized关键字时，线程1获取到了锁，则线程2等待，线程1阻塞，则线程2一直等待；Lock就不会一直等待下去
5. synchronized为可重入锁，不可以中断，非公平；Lock，可重入锁，可以中断，通过构造方法可以设置公平与非公平模式
6. synchronized适用于少量同步代码块问题，Lock适用于锁大量同步代码块

### 3. 生产者消费者问题

`线程通信问题：`生产者消费者问题。生产线程操作完毕后通知消费线程去消费，不满足生产条件时 生产线程处于等待状态。消费线程消费完成后，通知生产线程。

**如果是多个线程之间通信，不进行合适的判断，则会出现虚假唤醒的情况**

> 详细虚假换新案例见 com/geek/demo03/ProductAndConsumer.java

#### 3.1 虚假唤醒

`虚假唤醒：`进程可以被唤醒，但是如果唤醒后不能被通知或者中断，这种唤醒被称作虚假唤醒。如果条件不满足则进程应该继续等待，这个等待的逻辑应该处于循环中，就会避免虚假唤醒的状况发生。

```java
while(num != 0){
    // 进入等待状态
    this.wait();
}
```

#### 3.2 Lock锁解决进程通信问题

```java
class Data2{
    private int num = 0;
    Lock lock = new ReentrantLock();
    Condition condition = lock.newCondition();

    // 生产操作
    void increment() throws InterruptedException {
        // Condition对象操作进程的等待和唤醒
        lock.lock();
        try {
            while(num != 0){
                // 进入等待状态
                condition.await();
            }
            num ++;
            System.out.println(Thread.currentThread().getName() + "-->" + num);
            // 通知其他线程，我生产完毕
            condition.signalAll();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    // 消费操作
    void decrement() throws InterruptedException {
        lock.lock();

        try {
            while(num == 0){
                // 进入等待状态
                condition.await();
            }
            num --;
            System.out.println(Thread.currentThread().getName() + "-->" + num);
            // 通知其他线程，我消费完毕
            condition.signalAll();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
}

```

#### 3.3 Lock锁实现多个线程之间唤醒指定线程

```java
@SuppressWarnings("all")
class Data3{
    private int num = 1;
    Lock lock = new ReentrantLock();
    Condition condition1 = lock.newCondition();
    Condition condition2 = lock.newCondition();
    Condition condition3 = lock.newCondition();

    void printA(){
        lock.lock();
        try {
            while (num != 1){
                condition1.await();
            }
            System.out.println(Thread.currentThread().getName() + "--->" + "AAA");
            num = 2;
            condition2.signal();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    void printB(){
        lock.lock();
        try {
            while (num != 2){
                condition2.await();
            }
            System.out.println(Thread.currentThread().getName() + "--->" + "BBB");
            num = 3;
            condition3.signal();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    void printC(){
        lock.lock();
        try {
            while (num != 3){
                condition3.await();
            }
            System.out.println(Thread.currentThread().getName() + "--->" + "CCC");
            num = 1;
            condition1.signal();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
}
```

### 4.线程8锁

什么是锁？锁的对象是什么？

锁的对象有两种，一种是对象（调用普通方法），一种是Class（调用static方法）

### 5.不安全集合类

#### 5.1List集合

在并发操作的情况下，写入ArrayList会出现ConcurrentModificationException 并发修改异常，因此ArrayList是线程不安全的。

```Java
public static void main(String[] args) {
    // ConcurrentModificationException，会出现并发修改异常
    List<String> list = new ArrayList<>();
    for (int i = 0; i < 10; i++) {
        new Thread(()->{
            list.add(UUID.randomUUID().toString().substring(0, 5));
            System.out.println(list.toString());
        }).start();
    }
}
```

**解决方案：**

1.使用Vector类来代替ArrayList

```java
List<String> list = new Vector<>();
```

`Vector`出现在JDK1.1中，而`ArrayList`出现在JDK1.2中，可见并不推荐开发者使用`Vector`类来代替`ArrayList`。`Vector`本质是使用`synchronized`关键字修饰代码块，效率比较低。

```java
// Vector的add()方法
public synchronized boolean add(E e) {
    modCount++;
    ensureCapacityHelper(elementCount + 1);
    elementData[elementCount++] = e;
    return true;
}
```

2.是ArrayList集合变成线程安全的

```java
Collections.synchronizedList(new ArrayList<>());
```

3.使用`JUC`并发集合`CopyOnWriteArrayList`实现

`CopyOnWriteArrayList`底层原理为**读写分离思想**。使用transient volatile关键字修饰Array，使用ReentrantLock对对象加锁，在写入前先复制一份，将元素插入到复制的list中，然后再将插入后的list写入到原来的list中。

```java
private transient volatile Object[] array;

public boolean add(E e) {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        Object[] elements = getArray();
        int len = elements.length;
        Object[] newElements = Arrays.copyOf(elements, len + 1);
        newElements[len] = e;
        setArray(newElements);
        return true;
    } finally {
        lock.unlock();
    }
}
```

#### 5.2Set集合

```java
public static void main(String[] args) {
    //Set<String> set = new HashSet<>();
    //Set<String> set = Collections.synchronizedSet(new HashSet<>());
    Set<String> set = new CopyOnWriteArraySet<>();

    for (int i = 0; i < 30; i++) {
        new Thread(()->{
            set.add(UUID.randomUUID().toString().substring(0,5));
            System.out.println(set);
        }).start();
    }
}
```

#### 5.3Map集合

```java
Map<String, String> map = new ConcurrentHashMap<>();
```

### 6.Callable接口

```java
public class CallableDemo {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        FutureTask futureTask = new FutureTask<>(new MyThread());
        new Thread(futureTask).start();
        Integer result = (Integer) futureTask.get();
        System.out.println(result);
    }
}

class MyThread implements Callable<Integer>{

    @Override
    public Integer call() throws Exception {
        System.out.println("调用call()方法...");
        return 1;
    }
}
```

### 7.常用辅助类

#### 7.1CountDownLatch

```java
//CountDownLatch减法计数器
public static void main(String[] args) throws InterruptedException {
    CountDownLatch countDownLatch = new CountDownLatch(5);
    for (int i = 0; i < 5; i++) {
        new Thread(()->{
            System.out.println(Thread.currentThread().getName() + " go out");
            //执行减 1 操作
            countDownLatch.countDown();
        },String.valueOf(i)).start();
    }
    //等待计数器归零，然后执行下面的代码。否则程序将阻塞在这里。
    countDownLatch.await();
    System.out.println("close door");
}
```

#### 7.2CyclicBarrier

```java
//加法计数器
//集齐七颗龙珠，满足条件后召唤神龙
public static void main(String[] args) {
    CyclicBarrier cyclicBarrier = new CyclicBarrier(7, () -> System.out.println("召唤神龙成功..."));

    for (int i = 0; i < 7; i++) {
        final int temp = i;
        new Thread(()->{
            System.out.println("集齐了" + temp + "颗龙珠");
            try {
                cyclicBarrier.await();
            } catch (InterruptedException | BrokenBarrierException e) {
                e.printStackTrace();
            }
        },String.valueOf(i)).start();
    }
}
```

#### 7.3Semaphopre

```java
//信号量
//需求：模拟抢车位游戏，6辆汽车抢占3个车位
public static void main(String[] args) {
    Semaphore semaphore = new Semaphore(3);

    for (int i = 1; i <= 6; i++) {
        final int temp = i;
        new Thread(()->{
            try {
                //得到资源
                semaphore.acquire();
                System.out.println("汽车" + temp + "抢到了车位...");
                TimeUnit.SECONDS.sleep(2);
                System.out.println("汽车" + temp + "离开了车位...");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }finally {
                //释放资源
                semaphore.release();
            }
        },"汽车" + i).start();
    }
}
```

**原理：**

`semaphore.acquire()`得到了资源，那些没有抢占到资源的线程进入等到状态

`semaphore.release()`释放资源，会将当前的信号量释放，然后唤醒等待的线程

**应用：**

- 多个线程互斥操作共享资源（A抢占到了共享资源，B就处于等待状态）
- 并发限流（控制最大线程数）

### 8.读写锁

#### 8.1没有使用读写锁

```java
@SuppressWarnings("all")
public class ReadWriteLockDemo {

    public static void main(String[] args) throws InterruptedException {
        MyMap myMap = new MyMap();
        for (int i = 1; i <= 6; i++) {
            final int temp = i;
            new Thread(()->{
                myMap.write(temp, UUID.randomUUID().toString().substring(0,5));
            },"线程" + i).start();
        }
        for (int i = 1; i <= 6; i++) {
            final int temp = i;
            new Thread(()->{
                myMap.read(temp);
            },"线程" + i).start();
        }

    }
}

class MyMap{
    volatile Map<Integer,String> map = new HashMap<>();

    public Map<Integer, String> getMap() {
        return map;
    }

    void read(Integer key){
        System.out.println(Thread.currentThread().getName() + "读取的key为：" + key);
        String result = map.get(key);
        System.out.println(Thread.currentThread().getName() + "读取key的结果为：" + result);
    }

    void write(Integer key,String value){
        System.out.println(Thread.currentThread().getName() + "执行写操作，key为：" + key + 		"，value为：" + value);
        map.put(key,value);
        System.out.println(Thread.currentThread().getName() + "执行写操作完毕");
    }
}
```

**在没有加读写锁的情况下会出现 写操作会出现插队的情况**

见程序输出结果

> 线程1读取的key为：1
> 线程2读取的key为：2
> 线程5读取的key为：5
> 线程5读取key的结果为：null
> 线程1读取key的结果为：null
> 线程2读取key的结果为：null
> 线程3读取的key为：3
> 线程6读取的key为：6
> 线程6读取key的结果为：null
> 线程4读取的key为：4
> 线程4读取key的结果为：null
> 线程3读取key的结果为：null
> 线程3执行写操作，key为：3，value为：ddbfe
> 线程6执行写操作，key为：6，value为：65a8c
> 线程6执行写操作完毕
> 线程2执行写操作，key为：2，value为：1fb69
> 线程2执行写操作完毕
> 线程1执行写操作，key为：1，value为：19e63
> 线程1执行写操作完毕
> 线程5执行写操作，key为：5，value为：e11d5
> 线程5执行写操作完毕
> 线程3执行写操作完毕
> 线程4执行写操作，key为：4，value为：18faa
> 线程4执行写操作完毕

从上面的输出结果可以观察到 *线程3执行写操作，key为：3，value为：ddbfe*，线程3执行了部分写操作后被线程6插进来，执行了线程6的写操作，出现了线程之间插队的情况。

#### 8.2使用读写锁

```java
@SuppressWarnings("all")
public class ReadWriteLockDemo2 {

    public static void main(String[] args) throws InterruptedException {
        //创建减法计数器对象
        CountDownLatch latch = new CountDownLatch(6);
        MyMap1 myMap = new MyMap1();

        for (int i = 1; i <= 6; i++) {
            final int temp = i;
            new Thread(()->{
                myMap.write(temp, UUID.randomUUID().toString().substring(0,5));
                //减法计数器减 1 
                latch.countDown();
            },"线程" + i).start();
        }
        // 此行代码以后的程序进入阻塞状态，直到减法计数器减到 0 为止，然后开始执行下面的代码
        latch.await();

        for (int i = 1; i <= 6; i++) {
            final int temp = i;
            new Thread(()->{
                myMap.read(temp);
            },"线程" + (i + 6)).start();
        }
    }
}

class MyMap1{

    volatile Map<Integer,String> map = new HashMap<>();
    //创建读写锁对象
    ReentrantReadWriteLock  readWriteLock = new ReentrantReadWriteLock();

    public Map<Integer, String> getMap() {
        return map;
    }

    void read(Integer key){
        //给读锁加锁
        readWriteLock.readLock().lock();

        try {
            System.out.println(Thread.currentThread().getName() + "读取的key为：" + key);
            String result = map.get(key);
            System.out.println(Thread.currentThread().getName() + "读取key的结果为：" + 	             result);
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            //给读锁解锁
            readWriteLock.readLock().unlock();
        }
    }

    void write(Integer key,String value){
        //给写锁加锁
        readWriteLock.writeLock().lock();
        try {
            System.out.println(Thread.currentThread().getName() + "执行写操作，key为：" + 				key + "，value为：" + value);
            map.put(key,value);
            System.out.println(Thread.currentThread().getName() + "执行写操作完毕");
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            //给写锁解锁
            readWriteLock.writeLock().unlock();
        }
    }
}
```

程序输出结果

> 线程5执行写操作，key为：5，value为：09193
> 线程5执行写操作完毕
> 线程1执行写操作，key为：1，value为：53164
> 线程1执行写操作完毕
> 线程2执行写操作，key为：2，value为：941be
> 线程2执行写操作完毕
> 线程6执行写操作，key为：6，value为：c1e11
> 线程6执行写操作完毕
> 线程4执行写操作，key为：4，value为：ed9f3
> 线程4执行写操作完毕
> 线程3执行写操作，key为：3，value为：94442
> 线程3执行写操作完毕
> 线程8读取的key为：2
> 线程8读取key的结果为：941be
> 线程10读取的key为：4
> 线程7读取的key为：1
> 线程11读取的key为：5
> 线程12读取的key为：6
> 线程12读取key的结果为：c1e11
> 线程9读取的key为：3
> 线程9读取key的结果为：94442
> 线程11读取key的结果为：09193
> 线程7读取key的结果为：53164
> 线程10读取key的结果为：ed9f3

从以上输出结构可以看出，写线程（线程1-6）在执行写操作时由于加了写锁，没有出现插队的情况。读线程（线程7-12）加了读锁，虽然出现了插队的情况，但是并不影响实际需求（允许这样的情况发生）

**读类型线程与写类型线程共存问题：**

- 两个读线程可以共存
- 读线程与写线程不能共存
- 写线程与写线程不能共存

**锁类型：**

- 独占锁（写锁）：一次只能被一个线程占有
- 共享锁（读锁）：一个可以被多个线程占有

### 9.阻塞队列

#### 9.1常见集合

#### 9.2应用场景

- 多线程的并发处理
- 线程池的处理

#### 9.3如何使用

- 阻塞队列添加线程
- 阻塞队列移除线程

#### 9.4四组API

| 方式         | 抛出异常  | 有返回值（不抛异常） | 阻塞等待 | 超时等待  |
| ------------ | --------- | -------------------- | -------- | --------- |
| 添加         | add()     | offer()              | put()    | offer(,,) |
| 移除         | remove()  | poll()               | take()   | poll(,)   |
| 检测队列元素 | element() | peek()               | 无       | -         |

#### 9.5同步队列

**SynchronousQueue同步队列**

特点：put一个元素后必须take该元素后才能 put下一个元素

```java
public static void main(String[] args) {

    SynchronousQueue<String> queue = new SynchronousQueue<>();

    new Thread(()->{
        try {
            System.out.println(Thread.currentThread().getName() + "put " + "1");
            queue.put("1");
            System.out.println(Thread.currentThread().getName() + "put " + "2");
            queue.put("2");
            System.out.println(Thread.currentThread().getName() + "put " + "3");
            queue.put("3");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    },"T1").start();

    new Thread(()->{
        try {
            System.out.println(Thread.currentThread().getName() + "take " + queue.take());
            TimeUnit.SECONDS.sleep(3);
            System.out.println(Thread.currentThread().getName() + "take " + queue.take());
            TimeUnit.SECONDS.sleep(3);
            System.out.println(Thread.currentThread().getName() + "take " + queue.take());
            TimeUnit.SECONDS.sleep(3);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

    },"T2").start();
}
```

### 10.线程池

#### 10.1思想

由于线程的创建和回收这两个过程是非常消耗性能的，利用池化思想，事先创建好一些线程放入线程池中，使用完之后线程池再回收线程，达到减小资源开销的目的。

#### 10.2线程池好处

- 减小资源开销
- 提高响应速度
- 方便管理线程（实现线程复用，控制最大并发数）

```java
public static void main(String[] args) {
    //创建只有单条线程的线程池
    //ExecutorService threadPool = Executors.newSingleThreadExecutor();
    //创建固定大小的线程池
    //ExecutorService threadPool = Executors.newFixedThreadPool(5);
    //创建可伸缩大小的线程池
    ExecutorService threadPool = Executors.newCachedThreadPool();

    try {
        for (int i = 0; i < 100; i++) {
            threadPool.execute(()->{
                System.out.println(Thread.currentThread().getName());
            });
        }
    } catch (Exception e) {

    } finally {
        //关闭线程池
        threadPool.shutdownNow();
    }
}
```

#### 10.3线程池7大参数

```java
public ThreadPoolExecutor(int corePoolSize,		//核心线程数量
                          int maximumPoolSize,  //最大线程数量
                          long keepAliveTime,   //超过该时间没人调用就释放资源
                          TimeUnit unit,		//超时单位
                          BlockingQueue<Runnable> workQueue, //阻塞队列
                          ThreadFactory threadFactory, //线程工厂，这个参数一般不动
                          RejectedExecutionHandler handler //拒绝策略) {
    if (corePoolSize < 0 ||
        maximumPoolSize <= 0 ||
        maximumPoolSize < corePoolSize ||
        keepAliveTime < 0)
        throw new IllegalArgumentException();
    if (workQueue == null || threadFactory == null || handler == null)
        throw new NullPointerException();
    this.corePoolSize = corePoolSize;
    this.maximumPoolSize = maximumPoolSize;
    this.workQueue = workQueue;
    this.keepAliveTime = unit.toNanos(keepAliveTime);
    this.threadFactory = threadFactory;
    this.handler = handler;
```

#### 10.4手动创建线程池

```java
public static void main(String[] args) {
    ThreadPoolExecutor threadPool = new ThreadPoolExecutor(
            2,
            5,
            3,
            TimeUnit.SECONDS,
            new LinkedBlockingQueue<>(3),
            Executors.defaultThreadFactory(),
            new ThreadPoolExecutor.DiscardOldestPolicy()
    );
    try {
        for (int i = 1; i <= 9; i++) {
            threadPool.execute(()->{
                System.out.println(Thread.currentThread().getName());
            });
        }
    } catch (Exception e) {

    } finally {
        //关闭线程池
        threadPool.shutdownNow();
    }
}
```

#### 10.5四种拒绝策略

```java
/**
 * 四种拒绝策略：
 * AbortPolicy：队列满了则抛出异常
 * CallerRunsPolicy：队列满了谁叫你调的我则 谁去执行(交个main线程去执行)
 * DiscardPolicy：队列满了，则直接抛弃，不会抛出异常
 * DiscardOldestPolicy：队列满了，则会判断最早执行的线程有没有空闲，如有空闲则交个它执行
 */
```

### 11.JMM

**概念：**JMM全称为java 内存模型，是一个不存在的约定。java内存模型分为：线程工作内存和主内存

**约定：**

- 线程解锁前，应该必须把自己工作内存的共享变量刷新回主存
- 线程加锁前，应该必须把主存中最新的共享变量应该立刻同步到线程的工作内存中
- 线程加锁和解锁必须保证是同一把锁

### 12.Volatile

**概念：**volatile是java虚拟机提供的轻量级的同步机制（synchronized为重量级同步机制，volatile没有synchronized强大）

**特点：**

- 保证可见性
- 不保证原子性
- 避免指令重排

**共享变量从主内存到线程的工作内存的8个步骤：**

<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200614103249316.png" alt="image-20200614103249316" style="zoom:70%;" />

如果线程B修改了主存中的共享变量，而线程A对主存共享变量没有可见性，没有及时同步主存中共享变量，就会导致数据不一致问题，如下图所示：

![image-20200614104226722](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200614104226722.png)

#### 12.1保证可见性

```java
/**
 * @author : 
 * @date : 2020/6/14
 * @description : Volatile关键字的使用（保证共享变量的内存可见性）
 *
 * 反例
 * private static int num = 0;
 * 由于num没有用volatile关键字修饰，在主存与线程的工作内存 不可见
 * 主线程将num重新赋值为2，但是由于线程1内存不可见的问题，不能同步到num值的变化，因此程序进入死循环状态\
 *
 * 正例
 * private static volatile int num = 0;
 * 解决思路：让线程1知道主存中的共享变量发生了变化
 * 解决方案：使用volatile关键字修饰 共享变量num
 */
@SuppressWarnings("all")
public class VolatileDemo {

    private static volatile int num = 0;

    public static void main(String[] args) throws InterruptedException {
        new Thread(()->{
            while (num == 0){
            }
        },"线程1").start();
        TimeUnit.SECONDS.sleep(3);

        num = 2;
        System.out.println("num值为：" + num);
    }
}
```

#### 12.2不保证原子性

即一个线程在执行过程中（8个步骤）是不能被打扰，不能被分割的，要么全部成功，要么全部失败。

```Java
/**
 * @author : 
 * @date : 2020/6/14
 * @description :
 * 线程原子性：线程在执行过程中不能被打断，要么全部成功，要么全部失败
 * volatile关键字不保证线程的原子性操作
 */
public class VolatileACIDDemo {

    private static volatile int num = 0;

    public static void main(String[] args) {
        //开启20条线程
        for (int i = 0; i < 20; i++) {
            //每条线程循环调用1000次 add()方法，预期结果 num = 20000
            new Thread(()->{
                for (int j = 0; j < 1000; j++) {
                    add();
                }
            }).start();
        }

        //java程序中最少有2条线程在执行（main线程和GC垃圾回收线程）
        //如果活跃的线程数大于2，说明除了main线程和GC线程外还有线程在工作，则进行礼让，即不执行while循环以下的程序
        while (Thread.activeCount() > 2){
            Thread.yield();
        }

        System.out.println("num值为 " + num);
    }

    private static void add() {
        num ++;
    }
}
```

如何实现不使用lock锁和synchronized关键字来保证线程的原子操作？

使用juc包中的原子类就可以实现。这些原子类底层是直接与操作系统交互的，直接在内存中修改值，方法都为原生的native方法。

```java
/**
 * @author : 赵静超
 * @date : 2020/6/14
 * @description :
 * 线程原子性：线程在执行过程中不能被打断，要么全部成功，要么全部失败
 * volatile关键字不保证线程的原子性操作
 *
 * 解决方案：
 *      要求不使用Lock锁和Synchronized关键字来实现线程的原子操作
 *      使用juc包中的原子类可以解决该问题
 */
public class VolatileACIDDemo {

    //private static volatile int num = 0;
    private static AtomicInteger num = new AtomicInteger(1);

    public static void main(String[] args) {
        //开启20条线程
        for (int i = 0; i < 20; i++) {
            //每条线程循环调用1000次 add()方法，预期结果 num = 20000
            new Thread(()->{
                for (int j = 0; j < 1000; j++) {
                    add();
                }
            }).start();
        }

        //java程序中最少有2条线程在执行（main线程和GC垃圾回收线程）
        //如果活跃的线程数大于2，说明除了main线程和GC线程外还有线程在工作，则进行礼让，即不执行while循环以下的程序
        while (Thread.activeCount() > 2){
            Thread.yield();
        }

        System.out.println("num值为 " + num);
    }

    private static void add() {
        //num ++;
        num.getAndIncrement();
    }
}
```

#### 12.3指令重排

**概念：**计算机并不是按照你写的程序的顺序去执行的

**指令重排过程：**源代码--->编译器优化的重排--->指令并行会重排--->内存系统也会重排--->程序执行

**指令重排约定：**编译器在进行指令重排的时候会考虑数据之间的依赖问题

> 举例：

```java
int x = 1; //1
int y = 2; //2
x = x + 5; //3
y = x * x; //4

//我们所期望的程序执行顺序是1 2 3 4顺序执行
//但是计算机的执行过程可能是2 1 3 4，也有可能是1 3 2 4
//但是绝对不可能出现3 1 4 2的这种情况，因为处理器首先会考虑数据之间的依赖问题，首先先声明变量x，才会进行 
// x = x + 5操作
```

> 变量a b x y初始值为0
>
> 正常的执行顺序及结果

| 线程A | 线程B |
| ----- | ----- |
| x = a | y = b |
| b = 1 | a = 1 |

正常的执行结果为：x = 0 ：y = 0;

> 但是由于指令重排机制，会出现异常的执行顺序

| 线程A | 线程B |
| ----- | ----- |
| b = 1 | a = 1 |
| x = a | y = b |

异常的执行结果为 x = 1; y = 1

**总结：**volatile关键字可以保证内存可见性，但是不保证线程的原子性操作（可以使用juc的原子类来保证线程的原子性操作），由于volatile的内存屏障机制，可以避免指令重排现象。

![volatile内存屏障](https://i.loli.net/2020/06/14/x8MqEwXGkJSPAgd.png)

### 13.深入理解CAS

![Unsafe类](https://i.loli.net/2020/06/14/wAHTMsJ397fz2GY.png)

**自旋锁：**

![自旋锁](https://i.loli.net/2020/06/14/PBVwAIzyDNirfk6.png)

**CAS过程：**比较当前工作内存中的值和主存中的值是否相等，如果相等是期望的值，则执行操作。

```java
AtomicInteger atomicInteger = new AtomicInteger(2020);

//int expect, int update
//expect为期望值，update为想要更新的值，如果期望值与初始值相等，则将想要更新的值赋值给初始值。
atomicInteger.compareAndSet(2020,2021);
System.out.println(atomicInteger.get());
```

**缺点：**

- 循环耗时
- 一次只能保证一个共享变量的原子性
- 会导致ABA问题

> ABA问题

<img src="https://i.loli.net/2020/06/14/vRhxiZ1NLVBaD6T.png" alt="ABA问题" style="zoom:50%;" />



线程1和线程2操作共享变量A，线程1期望A是1然后希望将其赋值为2。此刻线程2先与线程1拿到共享变量A的值，将其赋值为3，之后由将其赋值为1，此刻共享变量的值为1。但是此刻共享变量的值已经不再是初始值的那个1了。此时线程1 不知道线程2已经改变了共享变量的值。**这就是ABA问题**。

线程2可以对操作共享变量的值，但是这个操作得让线程1知道。（联想MySQL表中的version字段，每次操作该记录，version值就加1）

### 14.原子引用

```java
/**
 * @author : Jeffersonnn
 * @date : 2020/6/14 17:22
 * @description :
 * CAS(compare and swap)即比较并交换
 */
@SuppressWarnings("all")
public class CASDemo {

    private static AtomicStampedReference<Integer> atomicReference = new AtomicStampedReference<>(1,1);

    public static void main(String[] args) {


        new Thread(()->{
            int stamp = atomicReference.getStamp();
            System.out.println("a ---> " + stamp);

            try {
                TimeUnit.SECONDS.sleep(3);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            atomicReference.compareAndSet(1, 2, atomicReference.getStamp(), atomicReference.getStamp() + 1);
            System.out.println("a1 ---> " + atomicReference.getStamp());
            atomicReference.compareAndSet(2, 1, atomicReference.getStamp(), atomicReference.getStamp() + 1);
            System.out.println("a2 ---> " + atomicReference.getStamp());

        },"a").start();

        new Thread(()->{
            int stamp = atomicReference.getStamp();
            System.out.println("b ---> " + stamp);

            try {
                TimeUnit.SECONDS.sleep(3);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            atomicReference.compareAndSet(1, 2, stamp, stamp + 1);
            System.out.println("b1 ---> " + atomicReference.getStamp());
        },"b").start();
    }
}
```

### 15.各种锁的理解

#### 15.1公平锁、非公平锁

- 公平锁：对资源的使用很公平，不能插队，保证先来后到的有序性。

  ```java
  public ReentrantLock(boolean fair) {
      sync = fair ? new FairSync() : new NonfairSync();
  }
  ```

- 非公平锁：对资源的使用不公平，可以插队（Lock锁和Synchronized关键字默认都是非公平锁实现的）

  ```java
  public ReentrantLock() {
      sync = new NonfairSync();
  }
  ```

#### 15.2可重入锁

​	**概念：**拿到了外面的锁后，就可以拿到里面的锁了（里面的锁是自动获得的）

​	<img src="https://i.loli.net/2020/06/14/46qgPeMGOlSoxdR.png" alt="可重入锁" style="zoom:50%;" />

#### 15.3自旋锁

```java
/**
 * @author : Jeffersonnn
 * @date : 2020/6/14 19:24
 * @description :
 * 自定义实现自旋锁
 */
public class SpinLockDemo {

    //传入一个Thread，初始值为null
    AtomicReference<Thread>  atomicReference = new AtomicReference<Thread>();

    //自旋锁加锁
    public void myLock(){
        System.out.println(Thread.currentThread().getName() + " lock");
        //获取当前的线程
        Thread thread = Thread.currentThread();
        while ( ! atomicReference.compareAndSet(null,thread)){

        }
    }
    //自旋锁解锁
    public void myUnLock(){
        System.out.println(Thread.currentThread().getName() + " unLock");
        //获取当前的线程
        Thread thread = Thread.currentThread();
        atomicReference.compareAndSet(thread,null);
    }
}
```