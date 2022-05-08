# 深入理解Java虚拟机

## 第2章 Java内存区域与内存溢出异常

### 2.1概述

在Java程序中，内存管理机制由Java虚拟机负责管理，但是也正因为Java程序员把控制内存的权力交给了虚拟机，一旦出现<font color="red">内存泄漏</font>、<font color="red">内存溢出</font>等方面的问题，如果不了解虚拟机是怎么样使用内存的，那排查问题和修正问题将是非常艰难的工作。

本章节将会介绍Java虚拟机的各个区域，<font color="red">讲解这些区域的作用，服务的对象以及可能产生的问题</font>。

### 2.2 运行时数据区域

根据JVM规范，JVM所管理的内存区域将会包括以下几个运行时数据区域，它们分别是：

1. 方法区（线程共享数据区）
2. 堆内存（线程共享数据区）
3. 虚拟机栈（线程隔离数据区）
4. 本地方法栈（线程隔离数据区）
5. 程序计数器（线程隔离数据区）

<img src="https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/image.2v4zu0awgmm0.webp" alt="image" style="zoom:67%;" />

#### 2.2.1 程序计数器

程序计数器是一块较小的内存空间，它可以看做是当前线程所执行的字节码的行号指示器。<font color="red">在Java虚拟机的概念模型中，字节码解释器工作时是需要改变这个计数器的值来确定下一条需要执行的字节码指令。</font>

由于Java虚拟机多线程的实现是通过多个线程轮流切换，抢占处理器执行时间来实现的。所以在任何一个时刻，任意一个处理器只能执行一个线程任务，因此线程切换后为了能够正确恢复到上一次执行任务的位置，线程内部需要程序计数器来保存当前执行位置。每个线程每部的程序计数器互相独立、互不影响。我们称这类内存区域为：<font color="red">线程私有内存区域</font>。

如果线程正在执行一个Java方法，则程序计数器保存的是当前正在执行的虚拟机字节码指令地址；如果线程执行的是本地（Native）方法，则程序计数器保存的值为空。

**程序计数器异常情况：**

<font color="red">程序计数器区域不会出现OutOfMemoryError情况。</font>

> Java虚拟机多线程实现原理：多个线程轮流切换，抢占处理器执行时间来实现的。

#### 2.2.2 Java虚拟机栈

<font color="red">与程序计数器一样，Java虚拟机栈也是线程私有的，它的生命周期和线程的生命周期相同。</font>虚拟机栈描述的是Java方法执行的线程内存模型：每个方法被执行时，Java虚拟机栈都会同步创建一个栈帧（Stack Frame）用于存储局部变量表、操作数栈、动态链接、方法出口等信息。<font color="red">每个方法被调用直至执行完毕的过程，就对应着一个栈帧从入栈到出栈的过程。</font>

局部变量表存放着编译器可知的各种Java虚拟机基本数据类型、对象应用、返回地址。这种数据类型在局部变量表中的存储空间以`局部变量插槽`的形式表示。局部变量表所需要的内存空间在编译器就已经完成分配。当进入一个方法时，这个方法在栈帧中所需要的局部变量表空间是确定的，在方法执行期间也不会改变局部变量表的大小（这里的大小指的是插槽的数量）。

**虚拟机栈异常情况：**

1. StackOverflowError异常

   当线程请求的栈的深度超过虚拟机所允许的最大栈的深度，就会抛出StackOverflowError异常。

2. OutOfMemoryError异常

   当栈空间扩展无法申请到足够的内存时就会抛出OutOfMemoryError异常。

#### 2.2.3 本地方法栈

本地方法栈与虚拟机栈提供的作用是非常相似的，其区别就是虚拟机栈执行的是Java方法，本地方法栈执行的是本地Native方法。

#### 2.2.4 Java堆

Java堆内存是JVM管理的最大的一块内存区域，并且<font color="red">堆内存是被所有线程共享的</font>，由虚拟机启动时创建。Java堆内存的唯一目的就是存放对象实例，<font color="red">Java程序中几乎所有的对象实例都是在堆内存上分配的</font>。

将Java堆内存细分的目的是为了更好地回收内存、或者更快地分配内存。

Java堆内存可以处于物理上不连续的内存空间，逻辑是它被视为连续的。但是对于大对象（数组对象）多数虚拟机出于实现简单、存储高效的考虑，很可能会要求大对象存储在连续的内存空间。

目前主流的Java虚拟机的堆内存都是可以扩展的（<font color="red">由参数-Xmx和-Xms设定</font>）

**异常情况：**

1. OutOfMemoryError异常

   如果对内存没有足够多的内存来分配对象，并且堆内存内存不能再扩展时，JVM将会抛出OutOfMemoryError异常。

#### 2.2.5 方法区

方法区（Method Area）与Java堆一样，是各个线程共享的内存区域，它用于存储已被虚拟机加载的`类型信息`、`常量`、`静态变量`、即时编译器编译后的`代码缓存`等数据。

垃圾收集行为在这个区域的 确是比较少出现的，这区域的内存回 收目标主要是针对常量池的回收和对类型的卸载。

#### 2.2.6 运行时常量池

运行时常量池（Runtime Constant Pool）是方法区的一部分。Class文件中除了有类的版本、字 段、方法、接口等描述信息外，还有一项信息是常量池表（Constant Pool Table），用于存放编译期生 成的各种`字面量`与`符号引用`，这部分内容将在类加载后存放到方法区的运行时常量池中。

**异常情况：**

既然运行时常量池是方法区的一部分，自然受到方法区内存的限制，当常量池无法再申请到内存时会抛出<font color="red">OutOfMemoryError异常。</font>

#### 2.2.7 直接内存

直接内存（Direct Memory）并不是虚拟机运行时数据区的一部分，也不是《Java虚拟机规范》中 定义的内存区域。但是这部分内存也被频繁地使用，而且也可能导致OutOfMemoryError异常出现

**异常情况：**

一般服务 器管理员配置虚拟机参数时，会根据实际内存去设置-Xmx等参数信息，但经常忽略掉直接内存，使得 各个内存区域总和大于物理内存限制（包括物理的和操作系统级的限制），从而导致动态扩展时出现<font color="red"> OutOfMemoryError异常</font>

### 2.3 HotSpot 虚拟机对象探秘

笔者以最常用的虚拟机HotSpot和最常用 的内存区域Java堆为例，深入探讨一下HotSpot虚拟机在Java堆中对象分配、布局和访问的全过程。

#### 2.3.1 对象的创建

1. 类加载

   当Java虚拟机遇到一条字节码new指令时，首先将去检查这个指令的参数是否能在常量池中定位到 一个类的符号引用，并且检查这个符号引用代表的类是否已被加载、解析和初始化过。如果没有，那 必须先执行相应的类加载过程。

2. 为新生对象分配内存

   在类加载检查通过后，接下来虚拟机将为新生对象分配内存。

   常见的分配的内存的方式有两种：指针碰撞方式、空闲列表方式

   1. <font color="red">指针碰撞方式</font>

      如果Java堆内存是规整的，所有被使用过的内存都被放在一 边，空闲的内存被放在另一边，中间放着一个指针作为分界点的指示器，那所分配内存就仅仅是把那 个指针向空闲空间方向挪动一段与对象大小相等的距离，这种分配方式称为“指针碰撞”（Bump The Pointer）。

   2. <font color="red">空闲列表方式</font>

      但如果Java堆中的内存并不是规整的，已被使用的内存和空闲的内存相互交错在一起，那 就没有办法简单地进行指针碰撞了，虚拟机就必须维护一个列表，记录上哪些内存块是可用的，在分 配的时候从列表中找到一块足够大的空间划分给对象实例，并更新列表上的记录，这种分配方式称 为“空闲列表”（Free List）。

3. 分派到的内存空间初始化零值

   内存分配完成之后，虚拟机必须将分配到的内存空间（但不包括对象头）都初始化为零值。这步操作保证了对象的实例字段 在Java代码中可以不赋初始值就直接使用，使程序能访问到这些字段的数据类型所对应的零值。

4. 为对象进行必要的设置

   Java虚拟机还要对对象进行必要的设置，例如这个对象是`哪个类的实例`、如何才能找到类的`元数据信息`、对象的`哈希码`、对象的`GC分代年龄`等信息。

5. 对象init（执行构造方法）

   new指令之后会接着执行 ()方法，按照程序员的意愿对对象进行初始化，这样一个真正可用的对象才算完全被构造出来。

#### 2.3.2 对象的内存布局

在HotSpot虚拟机里，对象在堆内存中的存储布局可以划分为三个部分：**对象头**（Header）、**实例数据**（Instance Data）和**对齐填充**（Padding）。

1. 对象头

   在HotSpot虚拟机中对象头部分包括两类信息，分别是：存储对象自身的运行时数据和类型指针

   1. 对象的运行时数据

      第一类是用于存储对象自身的运行时数据，如哈希码（HashCode）、GC分代年龄、锁状态标志、线程持有的锁、偏向线程ID、偏向时间戳等，官方称之为**Mark Word**。

   2. 类型指针

      对象头的另外一部分是类型指针，即对象指向它的类型元数据的指针，Java虚拟机通过这个指针 来确定该对象是哪个类的实例。

2. 实例数据

   来实例数据部分是对象真正存储的有效信息，即我们在程序代码里面所定义的各种类型的字段内容。

3. 对齐填充

   对象的第三部分是对齐填充，这并不是必然存在的，也没有特别的含义，它仅仅起着占位符的作用。（HotSpot虚拟机要求对象起始地址必须是8字节的整数倍）

#### 2.3.3 对象的访问定位

创建对象自然是为了后续使用该对象，我们的Java程序会通过栈上的reference数据来操作堆上的具 体对象。主流的访问方式主要有使用**句柄**和**直接指针**两种。

1. 句柄访问

   1. 实现方式

      使用句柄访问的话，Java堆中将可能会划分出一块内存来作为句柄池，reference中存储的就 是对象的句柄地址，而句柄中包含了**对象实例数据**与**类型数据**各自具体的地址信息。

      ![image](https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/image.2f5dhrryarwg.webp)

   2. 优点

      使用句柄来访问的最大好处就是reference中存储的是**稳定句柄地址**，在对象被移动（垃圾收集时移动对象是非常普遍的行为）时只会改变句柄中的实例数据指针，而 reference本身不需要被修改

2. 直接指针访问

   1. 实现方式
   
      使用直接指针访问的话，reference中存储的直接就是对象地址。
   
      ![image](https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/image.2f5dhrryarwg.webp)
   
   2. 优点
   
      使用直接指针来访问最大的好处就是**速度更快**，它节省了一次指针定位的时间开销

### 2.4 实战：OutOfMemoryError异常

#### 2.4.1 Java堆溢出

Java堆用于储存对象实例，我们只要不断地创建对象，并且保证GC Roots到对象之间有可达路径 来避免垃圾回收机制清除这些对象，那么随着对象数量的增加，总容量触及最大堆的容量限制后就会 产生内存溢出异常。

将堆的最小值-Xms参数与最大值-Xmx参数 设置为一样即可避免堆自动扩展

通过参数-XX：+HeapDumpOnOutOf-MemoryError可以让虚拟机 在出现内存溢出异常的时候Dump出当前的内存堆转储快照以便进行事后分析

```java
/**
 * @author: lucas.zhao@kuhantech.com
 * @date: 2022/4/28 21:29
 * @description: VM args: -Xms20m -Xmx20m -XX:+HeapDumpOnOutOfMemoryError
 */
public class HeapOOM {
    static class OOMObject {
    }

    public static void main(String[] args) {
        ArrayList<OOMObject> list = new ArrayList<OOMObject>();
        while(true) {
            list.add(new OOMObject());
        }
    }
}
```

```shell
# 运行日志
Connected to the target VM, address: '127.0.0.1:6695', transport: 'socket'
java.lang.OutOfMemoryError: Java heap space
Dumping heap to java_pid12708.hprof ...
Heap dump file created [28323297 bytes in 0.052 secs]
```

#### 2.4.2 虚拟机栈和本地方法栈溢出

**栈容量只能由-Xss参数来设定**

关于虚拟 机栈和本地方法栈，在《Java虚拟机规范》中描述了两种异常：

1. 如果线程请求的栈深度大于虚拟机所允许的最大深度，将抛出StackOverflowError异常。
   1. 如果虚拟机的栈内存允许动态扩展，当扩展栈容量无法申请到足够的内存时，将抛出 OutOfMemoryError异常。


```java
/**
 * @date: 2022/5/2 16:40
 * @description: -Xss128k
 * 栈溢出
 */
public class JavaVMStackSOF {
    private static int num = 1;
    private static void test() {
        num ++;
        test();
    }
    public static void main(String[] args) {
        try {
            test();
        } catch (Throwable exception) {
            System.out.println("num:" + num);
            throw exception;
        }
    }
}
```

```log
num:1087
Exception in thread "main" java.lang.StackOverflowError
	at com.geek.jvm.JavaVMStackSOF.test(JavaVMStackSOF.java:11)
```



```java
/**
 * @author: lucas.zhao@kuhantech.com
 * @date: 2022/5/2 16:58
 * @description: 创建线程导致栈内存溢出
 */
public class JavaVMStackOOM {
    private void dontStop() {
        while (true) {

        }
    }
    private void stackLeakByThread() {
        while (true) {
            new Thread(new Runnable() {
                @Override
                public void run() {
                    dontStop();
                }
            }).start();
        }
    }
    public static void main(String[] args) {
        JavaVMStackOOM stackOOM = new JavaVMStackOOM();
        stackOOM.stackLeakByThread();
    }
}
```

#### 2.4.3 方法区和运行时常量池溢出

#### 2.4.4 本机内存直接溢出

直接内存（Direct Memory）的容量大小可通过<font color="red">-XX：MaxDirectMemorySize</font>参数来指定，如果不去指定，则默认与Java堆最大值（由-Xmx指定）一致，代码清单2-10越过了DirectByteBuffer类直接通 过反射获取Unsafe实例进行内存分配

```java
/**
 * @author: lucas.zhao@kuhantech.com
 * @date: 2022/5/2 17:18
 * @description: -Xmx20M -XX:MaxDirectMemorySize=10M
 */
public class DirectMemoryOOM {
    private static final int _1MB = 1024 * 1024;

    public static void main(String[] args) throws IllegalAccessException {
        Field unsafeField = Unsafe.class.getDeclaredFields()[0];
        unsafeField.setAccessible(true);
        Unsafe unsafe = (Unsafe) unsafeField.get(null);
        while (true) {
            unsafe.allocateMemory(_1MB);
        }
    }
}
```

```shell
Exception in thread "main" java.lang.OutOfMemoryError
	at sun.misc.Unsafe.allocateMemory(Native Method)
	at com.geek.jvm.DirectMemoryOOM.main(DirectMemoryOOM.java:20)
Disconnected from the target VM, address: '127.0.0.1:14932', transport: 'socket'
```

## 第3章 垃圾收集器与内存分配策略

### 3.1 概述

垃圾收集需要完成的三件事情：

1. 哪些内存需要回收?
2. 什么时候回收？
3. 如何回收？

其中程序计数器、Java虚拟机栈、本地方法栈三个内存区域都是随着线程而生、随线程而死。栈中的栈帧随着方法的进入和退出有条不紊得进行着入栈和出栈的操作。每一个栈帧需要分配多少内存基本上类结构确定下来时就已经知道了。因此这几个区域的内存的分配和回收都是具备确定性的，当方法结束或者线程结束时，内存自然就跟着回收了。

然而Java堆和方法区这两个内存区域具有很显著的不确定型。同一个接口的不同的实现类需要的内存可能不一样，一个方法执行不同的分支所需要的内存可能不一样。所以只有在运行时期，我们才知道程序究竟会创建哪些对象、创建多少个对象。这部分内存的分配和回收是动态的，垃圾收集器所关注的就是这部分内存的管理。

### 3.2 对象已死

堆内存里面放着Java程序中几乎所有对象的实例，垃圾收集器在堆进行回收之前会首先判断对象是存活，还是死亡（死去的对象即不可能被任何途径引用的对象）

#### 3.2.1 引用计数算法（<font color="red">不被Java虚拟机使用</font>）

在对象中添加一个引用计数器，每当有一个地方 引用它时，计数器值就加一；当引用失效时，计数器值就减一；任何时刻计数器为零的对象就是不可 能再被使用的。

虽然引用计数算法占用了一些额外的空间来进行计数，但是它原理简单，并且判定效率也很高，在大多数情况下是一个不错的算法。<font color="red">但是在目前主流的Java虚拟机中都没有选用引用计数算法来管理内存</font>。主要原因是需要考虑很多例外的情况，必须要配合大量额外的处理才能正确的工作。<font color="red">譬如单纯的引用计数 就很难解决对象之间相互循环引用的问题。</font>

代码实例：objA和objB这两个对象再无任何引用，实际上这两个对象已 经不可能再被访问，但是它们因为互相引用着对方，导致它们的引用计数都不为零，引用计数算法也 就无法回收它们。

```java
public class ReferenceCountingGC {
    public Object instance = null;
    private static final int _1MB = 1024 * 1024;
    /**
     * 这个成员属性的唯一意义就是占点内存，以便能在GC日志中看清楚是否有回收过
     */
    private byte[] bigSize = new byte[2 * _1MB];
    public static void testGC() {
        ReferenceCountingGC objA = new ReferenceCountingGC();
        ReferenceCountingGC objB = new ReferenceCountingGC();
        objA.instance = objB;
        objB.instance = objA;
        objA = null;
        objB = null;
        // 假设在这行发生GC，objA和objB是否能被回收？
        System.gc();
    }
}
```

#### 3.2.2 可达性分析算法

当前主流的商用程序语言的内存管理子系统，都是通过可达性分析（Reachability Analysis）算法来判定对象是否存活的。这个算法的基本思路就是通过 一系列称为“GC Roots”的根对象作为起始节点集，从这些节点开始，根据引用关系向下搜索，搜索过程所走过的路径称为“引用链”，如果某个对象到GC Roots间没有任何引用链相连，即该对象到GC Roots不可达，证明该对象是不可能再被使用的。

![image](https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/image.1zzsadjc1rc0.webp)

#### 3.2.1 再谈引用

JDK1.2之后对对象的引用进行了扩充，分别是强引用、软引用、弱引用、虚引用。四种引用类型依次减弱

1. 强引用（Strongly Reference）

   强引用是在代码程序中普遍存在引用赋值，即`Object obj=new Object()`这种引用关系。只要强引用关系存在，垃圾收集器就永远不会回收强引用对象。

2. 软引用（Soft Reference）

   软引用是用来描述一些还有用，但非必须的对象。

   只被软引用关联着的对象，在系统将要发生内 存溢出异常前，会把这些对象列进回收范围之中进行第二次回收，如果这次回收还没有足够的内存， 才会抛出内存溢出异常。

3. 弱引用（Weak Reference）

   弱引用也是用来描述那些非必须对象，但是它的强度比软引用更弱一些。

   被弱引用关联的对象只 能生存到下一次垃圾收集发生为止。当垃圾收集器开始工作，无论当前内存是否足够，都会回收掉被弱引用关联的对象。

4. 虚引用（Phantom Reference）

   虚引用也称为“幽灵引用”或者“幻影引用”，它是最弱的一种引用关系。

   一个对象是否有虚引用的存在，完全不会对其生存时间构成影响。为一个对象设置虚 引用关联的唯一目的只是为了能在这个对象被收集器回收时收到一个系统通知。

#### 3.2.1 生存还是死亡

即使在可达性分析算法中判定为不可达的对象，也不是“非死不可”的，这时候它们暂时还处于“缓 刑”阶段。要真正宣告一个对象死亡，至少要经历两次标记过程：如果对象在进行可达性分析后发现没有与GC Roots相连接的引用链，那它将会被第一次标记。随后进行一次筛选，筛选的条件是此对象是 否有必要执行finalize()方法。如果一个对象确实有必要执行finalize()方法，那么垃圾收集器认为这个对象该死，可以被回收了。

#### 3.2.1 回收方法区

### 3.3 垃圾收集算法

#### 3.3.1 分带收集理论

分带垃圾收集理论的两个分带假说：

1. 弱分带假说

   绝大多数对象都是朝生熄灭的

2. 强分带假说

   熬过越多次垃圾收集过程的对象就越难以消 亡。

现代垃圾收集器的设计原则：收集器应该将Java堆划分成不同的区域，然后将回收对象依据其年龄（年龄即对象熬过垃圾收集过程的次数）分配到不同的区 域之中存储。

堆收集名词解释：

1. 新生代收集（Minor GC/Young GC）

   指目标只是新生代的垃圾收集

2. 老年代收集（Major GC/Old GC）

   指目标只是老年代的垃圾收集。目前只有CMS收集器会有单 独收集老年代的行为。

3. 混合收集（Mixed GC）

   指目标是收集整个新生代以及部分老年代的垃圾收集。目前只有G1收 集器会有这种行为。

4. 整堆收集（Full GC）

   收集整个Java对和方法区的垃圾收集

#### 3.3.2 标记-清除算法

标记清除算法分为"标记"和“清除”两个过程，首先标记出需要清除的对象，在标记完成之后，统一回收掉被标记的对象。标记的过程就是对象是否属于垃圾的判定过程。

![image-20220508152432639](https://s2.loli.net/2022/05/08/sgRpFzP3O4KGBth.png)

**<font color="red">缺点：</font>**

1. 算法执行效率不稳定

   如果Java堆中包含大量对 象，而且其中大部分是需要被回收的，这时必须进行大量标记和清除的动作，导致标记和清除两个过 程的执行效率都随对象数量增长而降低

2. 会出现内存空间碎片化的问题

   标记、清除之后会产生大 量不连续的内存碎片。空间碎片太多可能会导致当以后在程序运行过程中需要分配较大对象时无法找 到足够的连续内存而不得不提前触发另一次垃圾收集动作

#### 3.3.3 标记-复制算法

> 标记复制算法是为了解决标记清除算法面对大量可回收对象执行效率低的问题

它将可用 内存按容量划分为大小相等的两块，每次只使用其中的一块。当这一块的内存用完了，就将还存活着 的对象复制到另外一块上面，然后再把已使用过的内存空间一次清理掉。

![image](https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/image.mpgpweij240.webp)

**缺点：**

1. 如果内存中多数对象都是存 活的，这种算法将会产生大量的内存间复制的开销
2. 将可用内存缩小为了原来的一半，空间浪费较多

**优点：**

1. 但对于多数对象都是可回收的情况，算法需要复 制的就是占少数的存活对象，这个时候就实现简单，运行效率高

把新生代分为一块较大的Eden空间和两块较小的 Survivor空间，每次分配内存只使用Eden和其中一块Survivor。发生垃圾搜集时，将Eden和Survivor中仍 然存活的对象一次性复制到另外一块Survivor空间上，然后直接清理掉Eden和已用过的那块Survivor空间。

#### 3.3.4 标记-整理算法



### 3.4 HotSpot的算法实现细节

### 3.5 经典垃圾收集器

### 3.6 低延时垃圾收集器

### 3.7 选择合适的垃圾收集器

### 3.8 实现：内存分配与回收策略

### 3.9 本章小节















<font color="red"></font>