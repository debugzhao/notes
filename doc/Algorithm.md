## 一、线性表

### 1.1 顺序表

### 1.2 链表

#### 1.2.1 单向链表

#### 1.2.2 双向链表

#### 1.2.3 链表的复杂度分析

#### 1.2.4 链表反转

##### 递归算法

h链表反转内部是用[递归算法](https://cloud.tencent.com/developer/article/1356049)解决的。递归算法主要分为两个阶段，第一阶段：递归分解任务；第二阶段：回归分治任务

<img src="https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/image.53od6spif3g0.webp" alt="image" style="zoom: 50%;" />

##### 链表反转代码

```java
/**
 * 对整个链表进行反转
 */
public void reverse() {
    if (isEmpty() || length == 1) {
        return;
    }
    reverse(head.next);
}

/**
 * 反转指定节点 currentNode，并且把反转后的节点返回
 * @param currentNode
 * @return
 */
public Node<T> reverse(Node<T> currentNode) {
    if (currentNode.next == null) {
        head.next = currentNode;
        return currentNode;
    }
    // 递归反转当前节点的下一个节点，返回值就是链表反转后当前节点的上一个节点
    Node<T> preNode = reverse(currentNode.next);
    // 让返回的节点的下一个节点变为当前节点
    preNode.next = currentNode;
    currentNode.next = null;
    return currentNode;
}
```

#### 1.2.5 快慢指针

##### 问题描述

快慢指针指的是定义两个移动速度不一样的指针，以此来制造出想要的差值，通过这个差值可以找到链表上相应的节点。一般情况下快指针的移动速度是慢指针的两倍。

##### 中间值问题	

利用快慢指针，我们把一个链表看成一个跑道，假设a的速度是b的两倍，那么当a跑完全程后，b刚好跑一半，以 此来达到找到中间节点的目的。

如下图，最开始，slow与fast指针都指向链表第一个节点，然后slow每次移动一个指针，fast每次移动两个指针。

<img src="https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/image.62eqh0kfvfg.webp" alt="image" style="zoom: 67%;" />

```java
public class FastSlowTest {
    public static void main(String[] args) {

        Node<String> first = new Node<>("1", null);
        Node<String> second = new Node<>("2", null);
        Node<String> third = new Node<>("3", null);
        Node<String> fourth = new Node<>("4", null);
        Node<String> fifth = new Node<>("5", null);
        Node<String> six = new Node<>("6", null);
        Node<String> seven = new Node<>("7", null);

        first.next = second;
        second.next = third;
        third.next = fourth;
        fourth.next = fifth;
        fifth.next = six;
        six.next = seven;

        String value = middleValue(first);
        System.out.println("中间值：" + value);
    }

    private static <T> T middleValue(Node<T> node) {
        // 1.定义两个指针
        Node<T> fastNode = node;
        Node<T> slowNode = node;

        // 2.使用两个指针遍历链表，当快指针指向的节点没有下一个节点时，此时遍历结束
        // 这时慢指针指向的节点就是中间值
        while (fastNode.next != null) {
            // 变化快慢指针的值
            fastNode = fastNode.next.next;
            if (fastNode == null) {
                throw new NullPointerException("当前链表长度为偶数个，不能取中间值");
            }
            slowNode = slowNode.next;
        }
        return slowNode.data;
    }


    @Data
    public static class Node<T> {
        private T data;
        private Node<T> next;

        public Node(T data, Node<T> next) {
            this.data = data;
            this.next = next;
        }
    }
}
```

##### 单向链表是否有环问题

> == 和 equals区别？

<img src="https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/image.71c59bhzhb80.webp" alt="image" style="zoom:67%;" />

**问题具体化：**

使用快慢指针的思想，把链表当做跑道。如果链表有环则该链表是环形跑道。在一个环形跑道上如果两个人有速度差，则他们迟早会相遇，就说明链表上有环。

```java
private static <T> boolean isCircle(Node<T> node) { 
    // 定义快慢指针
    Node<T> fastNode = node;
    Node<T> slowNode = node;

    // 判断链表是否有环
    while (fastNode != null && fastNode.next != null) {
        fastNode = fastNode.next.next;
        slowNode = slowNode.next;
        if (fastNode.data.equals(slowNode.data)) {
            return true;
        }
    }
    return false;
}
```

##### 有环链表入口问题

当快慢指针相遇时，我们可以判断到链表中有环，这时重新设定一个新指针指向链表的起点，且步长与慢指针一样 为1，则慢指针与“新”指针相遇的地方就是环的入口。证明这一结论牵涉到数论的知识，这里略，只讲实现。

<img src="https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/image.34jwp9x4e2k0.webp" alt="image" style="zoom: 67%;" />

```java
private static <T> Node<T> getEntranceNode(Node<T> firstNode) {
    // 定义快慢指针
    Node<T> fastNode = firstNode;
    Node<T> slowNode = firstNode;
    Node<T> tempNode = null;

    // 判断链表是否有环
    while (fastNode != null && fastNode.next != null) {
        fastNode = fastNode.next.next;
        slowNode = slowNode.next;

        if (fastNode.equals(slowNode)) {
            tempNode = firstNode;
            continue;
        }
        if (tempNode != null) {
            tempNode = tempNode.next;
            if (tempNode.equals(slowNode)) {
                return tempNode;
            }
        }
    }
    return tempNode;
}
```

#### 1.2.6 循环链表

#### 1.2.7 约瑟夫问题

### 1.3 栈

#### 1.3.1 栈概述

栈是一种基于先进后出(FILO)的数据结构，是一种只能在一端进行插入和删除操作的特殊线性表。它按照先进后出 的原则存储数据，先进入的数据被压入栈底，最后的数据在栈顶，需要读数据的时候从栈顶开始弹出数据（最后一 个数据被第一个读出来）。

我们称数据进入到栈的动作为**压栈**，数据从栈中出去的动作为**弹栈**。

<img src="https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/image.3kr0fucqbrq0.webp" alt="image" style="zoom: 67%;" />

#### 1.3.2 栈的实现

```java
public void push(T data) {
    Node<T> oldFirstNode = head.next;
    head.next = new Node<>(data, oldFirstNode);
    this.length ++;
}

public T pop() {
    if (isEmpty()) {
        return null;
    }
    Node<T> oldFirstNode = this.head.next;
    this.head.next = oldFirstNode.next;
    this.length --;
    return oldFirstNode.data;
}
```

#### 1.3.3 案例

##### 括号匹配问题

1. 问题描述

   给定一个字符串，里边可能包含"()"小括号和其他字符，请编写程序检查该字符串的中的小括号是否成对出现。

   ```java
   // 例如：
   "(上海)(长安)"：正确匹配
   "上海((长安))"：正确匹配
   "上海(长安(北京)(深圳)南京)":正确匹配
   "上海(长安))"：错误匹配
   "((上海)长安"：错误匹配
   ```

2. 分析

   ```java
   1.创建一个栈用来存储左括号
   2.从左往右遍历字符串，拿到每一个字符
   3.判断该字符是不是左括号，如果是，放入栈中存储
   4.判断该字符是不是右括号，如果不是，继续下一次循环
   5.如果该字符是右括号，则从栈中弹出一个元素t；
   6.判断元素t是否为null，如果不是，则证明有对应的左括号，如果不是，则证明没有对应的左括号
   7.循环结束后，判断栈中还有没有剩余的左括号，如果有，则不匹配，如果没有，则匹配
   ```

3. 示例代码

   ```java
   public static boolean isMatch(String string) {
       // 1.创建栈对象，用来存储左括号
       Stack<String> myStack = new Stack<>();
       if (string.length() == 0) {
           return false;
       }
       // 2.从左往右遍历字符串
       for (int i = 0; i < string.length(); i++) {
           String currentChar = string.charAt(i) + "";
           // 3.判断当前字符串是否为左括号， 如果是则入栈
           if (currentChar.equals(LEFT_BRACKETS)) {
               myStack.push(currentChar);
           } else if (currentChar.equals(RIGHT_BRACKETS)){
               // 4.判断当前字符串是否为右括号，如果是则从当前栈中弹出一个左括号，并判断是否为null，如果为null则说明括号不匹配，返回false
               String data = myStack.pop();
               if (data == null) {
                   return false;
               }
           }
       }
       // 5.循环结束后，判断栈中是否还有剩余的左括号，如果有则说明括号不匹配，返回false
       return myStack.isEmpty();
   }
   ```

##### 逆波兰表达式求值问题

逆波兰表达式求值问题是我们计算机中经常遇到的一类问题，要研究明白这个问题，首先我们得搞清楚什么是逆波 兰表达式？要搞清楚逆波兰表达式，我们得从中缀表达式说起。

**中缀表达式：**

中缀表达式就是我们平常生活中使用的表达式，例如：1+3*2,2-(1+3)等等，中缀表达式的特点是：二元运算符总 是置于两个操作数中间。

**逆波兰表达式(后缀表达式)：**

逆波兰表达式是波兰逻辑学家J・卢卡西维兹(J・ Lukasewicz)于1929年首先提出的一种表达式的表示方法，后缀表 达式的特点：运算符总是放在跟它相关的操作数之后。

| 中缀表达式 | 后缀表达式 |
| :--------: | :--------: |
|    a+b     |    ab+     |
|  a+(b-c)   |   abc-+    |
| a*(b-c)+d  |  abc-*d+   |

**需求：**

给定一个只包含加减乘除四种运算的逆波兰表达式的数组表示方式，求出该逆波兰表达式的结果。

**分析：**

```java
1.创建一个栈对象oprands存储操作数
2.从左往右遍历逆波兰表达式，得到每一个字符串
3.判断该字符串是不是运算符，如果不是，把该该操作数压入oprands栈中
4.如果是运算符，则从oprands栈中弹出两个操作数o1,o2
5.使用该运算符计算o1和o2，得到结果result
6.把该结果压入oprands栈中
7.遍历结束后，拿出栈中最终的结果返回
```

**代码实现：**

```java
private static Double calculate(String[] notations) {
    // 1.定义一个操作数，用来存储操作数
    Stack<Double> myStack = new Stack<>();
    double tempResult = 0.0;
    double num1 = 0.0;
    double num2 = 0.0;

    // 2.从左至右遍历逆波兰表达式，得到每一个元素
    for (int i = 0; i < notations.length; i++) {
        String notation = notations[i];
        // 3.判断当前元素时操作数还是操作符
        switch (notation) {
            case "+" : // 4.如果是操作符号，则从栈中弹出两个元素计算结果后，再压栈
                num1 = myStack.pop();
                num2 = myStack.pop();
                tempResult = num2 + num1;
                myStack.push(tempResult);
                break;
            case "-" :
                num1 = myStack.pop();
                num2 = myStack.pop();
                tempResult = num2 - num1;
                myStack.push(tempResult);
                break;
            case "*" :
                num1 = myStack.pop();
                num2 = myStack.pop();
                tempResult = num2 * num1;
                myStack.push(tempResult);
                break;
            case "/" :
                num1 = myStack.pop();
                num2 = myStack.pop();
                tempResult = num2 / num1;
                myStack.push(tempResult);
                break;
            default: // 5.如果是操作数，则直接压栈
                Double value = new Double(notation);
                myStack.push(value);
                break;
        }
    }
    // 6.循环完毕后，栈中最后一个元素就是计算结果
    long length = myStack.length();
    System.out.println("此时栈中元素个数为：" + length);
    if (length == 1) {
        return myStack.pop();
    } else {
        return null;
    }
}
```

### 1.4 队列

队列是一种基于先进先出(FIFO)的数据结构，是一种只能在一端进行插入,在另一端进行删除操作的特殊线性表，它 按照先进先出的原则存储数据，先进入的数据，在读取数据时先读被读出来。

<img src="https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/image.5kf2bml8qmg0.webp" alt="image" style="zoom:67%;" />

#### 1.4.1 队列API设计

#### 1.4.2 队列的实现

```java
public void enqueue(T data) {
    Node<T> newNode = new Node<>(data, null);
    if (this.last == null) {
        this.last = newNode;
        this.head.next = this.last;
    } else {
        Node<T> oldLast = this.last;
        this.last = newNode;
        oldLast.next = this.last;
    }
    this.length ++;
}

public T dequeue() {
    T data;
    if (isEmpty()) {
        data = null;
    } else if (this.length == 1) {
        Node<T> oldFirst = this.head.next;
        this.head.next = null;
        this.last = null;
        data = oldFirst.data;
    } else {
        Node<T> oldFirst = this.head.next;
        this.head.next = oldFirst.next;
        oldFirst.next = null;
        data = oldFirst.data;
    }
    this.length --;
    return data;
}
```

## 二、符号表

符号表最主要的目的就是将一个键和一个值联系起来，符号表能够将存储的数据元素是一个键和一个值共同组成的 键值对数据，我们可以根据键来查找对应的值。

<img src="https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/image.1tf2tcvpmedc.webp" alt="image" style="zoom:67%;" />

### 2.1符号表API设计

### 2.2符号表实现

```java
public void put(K key, V value) {
    if (key == null) {
        throw new IllegalArgumentException("The key value cannot be null.");
    }
    // 1.如果符号表中已经存在了键为K的键值对，则找到并替换
    Node<K, V> node = this.head;
    while (node.next != null) {
        node = node.next;
        if (node.key.equals(key)) {
            node.value = value;
            return;
        }
    }
    // 2.不存在则创建新的节点，头插法，链表长度加一
    Node<K, V> newNode = new Node<>(key, value, null);
    Node<K, V> oldFirstNode = head.next;
    this.head.next = newNode;
    newNode.next = oldFirstNode;
    this.length ++;
}

public V get(K key) {
    Node<K, V> tempNode = this.head;
    while (tempNode.next != null) {
        tempNode = tempNode.next;
        if (tempNode.key.equals(key)) {
            return tempNode.value;
        }
    }
    return null;
}

public void delete(K key) {
    if (this.isEmpty()) {
        throw new IllegalArgumentException("The current symbol table is empty, cannot be deleted");
    }
    Node<K, V> node = this.head;
    while (node.next != null) {
        if (node.next.key.equals(key)) {
            node.next = node.next.next;
            this.length --;
            return;
        }
        node = node.next;
    }
}
```

### 2.3有序符号表实现

## 三、二叉树入门

之前我们实现的符号表中，不难看出，符号表的增删查操作，随着元素个数N的增多，其耗时也是线性增多的，时 间复杂度都是O(n),为了提高运算效率，接下来我们学习树这种数据结构。

### 3.1 树的基本定义

树是我们计算机中非常重要的一种数据结构，同时使用树这种数据结构，可以描述现实生活中的很多事物，例如家 谱、单位的组织架构、等等。

树是由n（n>=1）个有限结点组成一个具有层次关系的集合。把它叫做“树”是因为它看起来像一棵倒挂的树，也就 是说它是根朝上，而叶朝下的。

### 3.2 树的相关术语

1. 节点的度
2. 叶子节点
3. 分支节点
4. 节点的层次
5. 节点的层序编号
6. 树的度
7. 树的高度
8. 森林
9. 孩子节点
10. 父节点
11. 兄弟节点

### 3.3 二叉树的基本定义

二叉树就是度不超过2的树（每个节点最多有两个子节点）

<img src="https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/image.rkc3yt225tc.webp" alt="image" style="zoom:67%;" />

##### 满二叉树

一个二叉树，如果每一个层的结点树都达到最大值，则这个二叉树就是满二叉树。

<img src="https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/image.24r7m474p3cw.webp" alt="image" style="zoom:80%;" />

##### 完全二叉树

叶子节点只能出现在最下层和次下层，并且最下层的节点只能出现在该层的最左边的树称为完全二叉树

<img src="https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/image.2v43xf0p7s00.webp" alt="image" style="zoom:80%;" />

### 3.4 二叉查找树的创建

#### 3.4.1 二叉树的节点类

#### 3.4.2 二叉查找树的API设计

#### 3.4.3 二叉查找树实现

#### 3.4.4 二叉查找树的其他便捷方法

##### 查找二叉树中最小的键

##### 查找二叉树中最大的键

### 二叉树的基础遍历

很多情况下，我们可能需要像遍历数组数组一样，遍历树，从而拿出树中存储的每一个元素，由于树状结构和线性 结构不一样，它没有办法从头开始依次向后遍历，所以存在如何遍历，也就是按照什么样的搜索路径进行遍历的问题

<img src="https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/image.51tb8xf3lzs0.webp" alt="image" style="zoom: 50%;" />

我们把树简单的画作上图中的样子，由一个根节点、一个左子树、一个右子树组成，那么按照根节点什么时候被访 问，我们可以把二叉树的遍历分为以下三种方式：

1. 前序遍历

   先访问根结点，然后再访问左子树，最后访问右子树

2. 中序遍历

   先访问左子树，中间访问根节点，最后访问右子树

3. 后序遍历

   先访问左子树，再访问右子树，最后访问根节点

如果我们分别对下面的树使用三种遍历方式进行遍历，得到的结果如下：

<img src="https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/image.1ggun6m65of4.webp" alt="image" style="zoom:50%;" />

#### 前序遍历

实现步骤

1. 把当前结点的key放入到队列中;
2. 找到当前结点的左子树，如果不为空，递归遍历左子树
3. 找到当前结点的右子树，如果不为空，递归遍历右子树

```java
/**
 * 前序遍历
 * @return 获取整个树中所有的键
 */
public Queue<K> preErgodic() {
    Queue<K> queue = new Queue<>();
    preErgodic(root, queue);
    return queue;
}

/**
 * 前序遍历
 * 获取指定树x中所有的键，并把所有的键放到keys队列中
 * @param x
 * @param keys
 */
public void preErgodic(Node x, Queue<K> keys) {
    // 递归方法出口
    if (x == null) {
        return;
    }
    // 把x节点的key放入队列keys中
    keys.enqueue(x.key);

    // 递归遍历x节点的左子树
    if(x.left != null) {
        preErgodic(x.left, keys);
    }

    // 递归遍历x节点的右子树
    if (x.right != null) {
        preErgodic(x.right, keys);
    }
}
```

#### 中序遍历

实现步骤：

1. 找到当前结点的左子树，如果不为空，递归遍历左子树
2. 把当前结点的key放入到队列中;
3. 找到当前结点的右子树，如果不为空，递归遍历右子树

```java
/**
 * 二叉树中序遍历
 * @return
 */
public Queue<K> middleErgodic() {
    Queue<K> queue = new Queue<>();
    middleErgodic(root, queue);
    return queue;
}

/**
 * 二叉树中序遍历
 * @param x
 * @param keys
 */
public void middleErgodic(Node x, Queue<K> keys) {
    if (x == null) {
        return;
    }
    // 递归遍历x节点的左子树
    if(x.left != null) {
        middleErgodic(x.left, keys);
    }
    // 把x节点的key放入队列keys中
    keys.enqueue(x.key);
    // 递归遍历x节点的右子树
    if (x.right != null) {
        middleErgodic(x.right, keys);
    }
}
```

#### 后续遍历

实现步骤：

1. 找到当前结点的左子树，如果不为空，递归遍历左子树 
2. 找到当前结点的右子树，如果不为空，递归遍历右子树
3. 把当前结点的key放入到队列中

```java
/**
 * 二叉树后序遍历
 * @return
 */
public Queue<K> afterErgodic() {
    Queue<K> queue = new Queue<>();
    afterErgodic(root, queue);
    return queue;
}

/**
 * 二叉树后序遍历
 * @param x
 * @param keys
 */
public void afterErgodic(Node x, Queue<K> keys) {
    if (x == null) {
        return;
    }
    // 递归遍历x节点的左子树
    if(x.left != null) {
        afterErgodic(x.left, keys);
    }
    // 递归遍历x节点的右子树
    if (x.right != null) {
        afterErgodic(x.right, keys);
    }
    // 把x节点的key放入队列keys中
    keys.enqueue(x.key);
}
```

#### 二叉树层序遍历

所谓的层序遍历，就是从根节点（第一层）开始，依次向下，获取每一层所有结点的值，有二叉树如下：

<img src="https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/image.he5cvx5zqv4.webp" alt="image" style="zoom:50%;" />

那么层序遍历的结果是：EBGADFHC

**实现步骤：**

1. 创建队列，存储每一层的结点
2. 使用循环从队列中弹出一个结点
   1. 获取当前结点的key
   2. 如果当前结点的左子结点不为空，则把左子结点放入到队列中
   3. 如果当前结点的右子结点不为空，则把右子结点放入到队列中

<img src="https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/image.4ieihdio3xu0.webp" alt="image" style="zoom:50%;" />

```java
/**
 * 使用层序遍历，获取整个树中所有的键
 * @return
 */
public Queue<K> layerErgodic() {
    // 定义两个队列，分别存储树中的键和树中的节点
    // 键队列
    Queue<K> keys = new Queue<>();
    // 节点队列
    Queue<Node> nodes = new Queue<>();

    // 刚开始往节点队列中放入根节点
    nodes.enqueue(root);

    while (!nodes.isEmpty()) {
        // 从节点队列中弹出一个节点，并且把该节点的key放入键队列中
        Node node = nodes.dequeue();
        keys.enqueue(node.key);
        // 判断该节点是否有左子节点，如果有则放入节点队列中
        if (node.left != null) {
            nodes.enqueue(node.left);
        }
        // 判断该节点是否有右子节点，如果有则放入节点队列中
        if (node.right!= null) {
            nodes.enqueue(node.right);
        }
    }
    return keys;
}
```

#### 二叉树的最大深度问题

**需求：**

给定一棵树，请计算树的最大深度（树的根节点到最远叶子结点的最长路径上的结点数）

<img src="https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/image.1dvswck09zr4.webp" alt="image" style="zoom:50%;" />

上面这课树的最大深度是4

**实现：**

1. 如果根节点为空，则最大深度为0
2. 计算左子树的最大深度
3. 计算右子树的最大深度
4. 当前树的最大深度是 = max(左子树的最大深度，右子树的最大深度)  + 1

```java
/**
 * 获取整个树的最大深度
 * @return
 */
public int maxDepth() {
    if (root == null) {
        return 0;
    }
    return maxDepth(root);
}

/**
 * 获取指定树node的最大深度
 * @param node
 * @return
 */
public int maxDepth(Node node) {
    if (node == null) {
        return 0;
    }

    // node节点的最大深度
    int nodeMaxDepth = 0;
    // node节点的左子树的最大深度
    int leftTreeMaxDepth = 0;
    // node节点的右子树的最大深度
    int rightTreeMaxDepth = 0;

    // 计算左子树的最大深度
    if (node.left != null) {
        leftTreeMaxDepth = maxDepth(node.left);
    }
    // 计算右子树的最大深度
    if (node.right != null) {
        rightTreeMaxDepth = maxDepth(node.right);
    }
    // 计算整个树的最大深度
    nodeMaxDepth = leftTreeMaxDepth >= rightTreeMaxDepth ? leftTreeMaxDepth + 1 : rightTreeMaxDepth + 1;
    return nodeMaxDepth;
}
```

#### 折纸问题

## 四、堆

### 4.1 堆的定义

堆是计算机科学中一类特殊的数据结构的统称，堆通常可以被看做是一棵<font color="red">完全二叉树的数组对象</font>。

#### 堆的特性：

1. 堆实际上是一个<font color="red">完全二叉树</font>

   它是完全二叉树，除了树的最后一层结点不需要是满的，其它的每一层从左到右都是满的，如果最后一层结点不 是满的，那么要求左满右不满

   <img src="https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/image.47045luzpm00.webp" alt="image" style="zoom: 50%;" />

2. 堆通常用<font color="red">数组实现</font>

   具体方法就是将二叉树的结点按照层级顺序放入数组中，根结点在位置1，它的子结点在位置2和3，而子结点的子 结点则分别在位置4,5,6和7，以此类推

   <img src="https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/image.6kdyqa0j3eg0.webp" alt="image" style="zoom: 50%;" />

   如果一个结点的位置为k，则它的父结点的位置为[k/2],而它的两个子结点的位置则分别为2k和2k+1。这样，在不 使用指针的情况下，我们也可以通过计算数组的索引在树中上下移动：从a[k]向上一层，就令k等于k/2,向下一层就 令k等于2k或2k+1。

3. 堆中的<font color="red">每个节点都大于等于它的两个子节点</font>

   > 此特性区别于二分查找树，二分查找树要求父节点 > 左子节点，父节点 < 右子节点

   每个结点都大于等于它的两个子结点。这里要注意堆中仅仅规定了<font color="red">每个结点大于等于它的两个子结点</font>，但这两个 子结点的顺序并没有做规定，跟我们之前学习的二叉查找树是有区别的

### 4.2堆的API设计

| 类名     | Heap                                                         |
| -------- | ------------------------------------------------------------ |
| 构造方法 | Heap(int capacity)：创建容量为capacity的Heap对象             |
| 成员方法 | 1.private boolean less(int i,int j)：判断堆中索引i处的元素是否小于索引j处的元素 <br/>2.private void exch(int i,int j):交换堆中i索引和j索引处的值<br/> 3.public T delMax():删除堆中最大的元素,并返回这个最大元素 <br/>4.public void insert(T t)：往堆中插入一个元素<br/> 5.private void swim(int k):使用上浮算法，使索引k处的元素能在堆中处于一个正确的位置<br/> 6.private void sink(int k):使用下沉算法，使索引k处的元素能在堆中处于一个正确的位置 |
| 成员变量 | 1.private T[] imtes : 用来存储元素的数组 <br/>2.private int N：记录堆中元素的个数 |

### 4.3 堆的实现

```java
public class Heap<T extends Comparable<T>> {

    /**
     * 存储堆中的元素
     */
    private T[] items;
    /**
     * 记录堆中元素的个数
     */
    private int count;

    public Heap(int capacity) {
        this.items = (T[]) new Comparable[capacity + 1];
        this.count = 0;
    }

    /**
     * 判断堆中索引i处的元素是否小于索引j处的元素
     * @param i
     * @param j
     * @return
     */
    public boolean less(int i, int j) {
        return items[i].compareTo(items[j]) < 0;
    }

    /**
     * 交换堆中i索引和j索引处的值
     * @param i
     * @param j
     */
    public void exchange(int i, int j) {
        T temp = items[i];
        items[i] = items[j];
        items[j] = temp;
    }

    /**
     * 往堆中插入一个元素
     * @param t
     */
    public void insert(T t) {
        items[++count] = t;
        swim(count);
    }

    /**
     * 使用上浮算法，使索引k处的元素能在堆中处于正确的位置
     * @param k 节点所在的索引
     */
    public void swim(int k) {
        // 通过循环，不断比较当前节点的值和其父节点的值的大小，如果当前节点的值 > 父节点的值，则交换两个元素位置
        // 数组是从下标1位置开始存储元素的
        while (k > 1) {
            if (less(k / 2, k)) {
                exchange(k / 2, k);
            }
            k = k / 2;
        }
    }

    /**
     * 删除堆中最大的元素，并返回这个元素
     * @return
     */
    public T deleteMax() {
        T max = items[1];
        // 交换索引1处的元素和最大索引处的元素，让完全二叉树的最下层最右侧的元素(即刚才索引最大处的元素)称为临时根节点
        exchange(1, count);
        // 删除当前最大索引处的元素，置为null即可
        items[count] = null;
        // 元素个数 -1
        count --;
        // 通过下沉算法调整当前临时根节点的位置，使堆重新有序
        sink(1);
        return max;
    }

    /**
     * 使用下沉算法，使索引k处的元素能在堆中处于正确的位置
     * @param k
     */
    public void sink(int k) {
        // 通过循环不断比较当前k节点和其左子节点2*k 以及右子节点2*k+1处的较大值的元素的大小，如果过当前节点小，则交换其位置
        while (2*k <= count) {
            // 获取当前节点的子节点的较大节点
            // 记录较大子节点所在的索引
            int tempMaxIndex = 0;
            // 如果当前节点存在右子节点，则进行比较
            if (2*k + 1 <= count) {
                tempMaxIndex = less(2*k + 1, 2*k) ? 2*k : 2*k + 1;
            } else {
                tempMaxIndex = 2*k;
            }

            // 比较当前节点和较大子节点的值
            if(less(tempMaxIndex, k)) {
                break;
            }
            exchange(k, tempMaxIndex);
            k = tempMaxIndex;
        }
    }
}
```

### 4.4 堆排序

#### 堆构造过程



#### 堆排序过程

对构造好的堆，我们只需要做类似堆删除的操作，就可以实现堆排序

1. 将堆定元素和堆中最后一个元素交换位置
2. 此时堆是无需状态的，这时需要对堆定元素下沉调整堆，把第二大的元素上浮到堆顶（此时原来堆中的最大的元素不参与调整，因为最大的元素已经到了数组最右边）
3. 重复第一步、第二部操作，直到堆中只剩下一个元素

```java
public class HeapSort {
    
    /**
     * 判断堆中索引i处的元素是否小于索引j处的元素
     * @param heap
     * @param i
     * @param j
     * @return
     */
    private static boolean less(Comparable[] heap, int i, int j) {
        return heap[i].compareTo(heap[j]) < 0;
    }

    /**
     * 交换heap堆中索引i和索引j处的值
     * @param heap
     * @param i
     * @param j
     */
    private static void exchange(Comparable[] heap, int i, int j) {
        Comparable tmp = heap[i];
        heap[i] = heap[j];
        heap[j] = tmp;
    }

    /**
     * 根据元数据source，构造出heap堆
     * @param source
     * @param heap
     */
    private static void createHeap(Comparable[] source, Comparable[] heap) {
        // 把source中的元素拷贝到heap数组中，此时heap是一个无序的堆
        System.arraycopy(source, 0, heap, 1, source.length);

        // 对堆中的元素做下沉调整（从堆长度的一半处开始，往索引1处扫描）
        for (int i = heap.length / 2; i > 0; i--) {
            sink(heap, i, heap.length - 1);
        }
    }

    /**
     * 对source数组中的数据从小大小尽心排序
     */
    public static void sort(Comparable[] source) {
        // 1.构建堆e
        Comparable[] heap = new Comparable[source.length + 1];
        createHeap(source, heap);
        // 2.定义一个变量，记录未排序的元素中的最大索引
        int num = heap.length - 1;
        // 3.通过循环，交换索引1处的元素和需要排序的元素列表中的最大索引处的元素
        while (num != 1) {
            // 交换元素
            exchange(heap, 1, num);
            // 4.排序交换后最大元素的索引，让它不要参
            // 5. 对索引1处的元素进行堆的下沉调整与对的下沉调整
            num --;
            sink(heap, 1, num);
        }
        // 把heap中的数据复制到原数组source中
        System.arraycopy(heap, 1, source, 0, source.length);
    }

    /**
     * 在heap堆中，对target处的元素做下沉，范围是 0~range
     * @param heap
     * @param target
     * @param range
     */
    private static void sink(Comparable[] heap, int target, int range) {
        // 通过循环不断比较当前k节点和其左子节点2*k 以及右子节点2*k+1处的较大值的元素的大小，如果过当前节点小，则交换其位置
        while (2*target <= range) {
            // 1.找出当前节点的最大的子节点
            // 最大子节点索引
            int tempMaxIndex = 0;
            if (2*target + 1 <= range) { // 如果target索引处的节点存在右子节点
                tempMaxIndex = less(heap, 2*target, 2*target + 1) ? 2*target + 1 : 2*target;
            } else {
                tempMaxIndex = 2*target; // 如果target索引处的节点不存在右子节点，则直接将左子节点的索引赋值给tempMaxIndex
            }

            // 2.比较当前节点的值和最大子节点的值
            if(!less(heap, target, tempMaxIndex)) {
                break;
            }
            exchange(heap, target, tempMaxIndex);
            target = tempMaxIndex;
        }
    }
}
```

## 五、优先队列

普通的队列是一种先进先出的数据结构，元素在队列尾追加，而从队列头删除。在某些情况下，我们可能需要找出 队列中的最大值或者最小值，例如使用一个队列保存计算机的任务，一般情况下计算机的任务都是有优先级的，我 们需要在这些计算机的任务中找出优先级最高的任务先执行，执行完毕后就需要把这个任务从队列中移除。普通的 队列要完成这样的功能，需要每次遍历队列中的所有元素，比较并找出最大值，效率不是很高，这个时候，我们就 可以使用一种特殊的队列来完成这种需求，优先队列。

<img src="C:\Users\lucas.zhao\AppData\Roaming\Typora\typora-user-images\image-20220327193327340.png" alt="image-20220327193327340" style="zoom:50%;" />

优先队列按照其作用不同，可以分为以下两种：

1. 最大优先队列

   可以获取并删除队列中最大的值

2. 最大优先队列

   可以获取并删除队列中最小的值

### 5.1 最大优先队列

我们之前学习过堆，而堆这种结构是可以方便的删除最大的值，所以，接下来我们可以基于堆区实现最大优先队 列。

#### API设计

| 类名     | MaxPriorityQueue<T>                                          |
| -------- | ------------------------------------------------------------ |
| 构造方法 | MaxPriorityQueue(int capacity)：创建容量为capacity的MaxPriorityQueue对象 |
| 成员方法 | 1.private boolean less(int i,int j)：判断堆中索引i处的元素是否小于索引j处的元素 <br/>2.private void exch(int i,int j):交换堆中i索引和j索引处的值 <br/>3.public T delMax():删除队列中最大的元素,并返回这个最大元素 <br/>4.public void insert(T t)：往队列中插入一个元素<br/> 5.private void swim(int k):使用上浮算法，使索引k处的元素能在堆中处于一个正确的位置 <br/>6.private void sink(int k):使用下沉算法，使索引k处的元素能在堆中处于一个正确的位置<br/>7.public int size():获取队列中元素的个数 8.public boolean isEmpty():判断队列是否为空 |
| 成员变量 | 1.private T[] imtes : 用来存储元素的数组<br/>2.private int N：记录堆中元素的个数 |

#### 代码实现

最大优先队列实际上就是堆，利用了`deleteMax `API实现了每次从堆中获取最大的元素

```java
/**
 * 上浮算法思想：使索引k处的元素能在堆中处于一个正确的位置
 * 上浮算法基本实现；通过循环比较当前元素和父元素的大小，如果当前元素比父元素大，则该数据往上走
 * @param k
 */
private void swim(int k) {
    while (k > 1) { // k = 1临界值问题考虑：如果K = 1时即为根节点，不需要再进行上浮，k = 1时不需要再进行循环
        if (less(k/2, k)) {
            exchange(k/2, k);
        }
        k = k / 2;
    }
}

/**
 * 下沉算法思想：使索引k处的元素能在堆中处于一个正确的位置
 * 下沉算法基本实现：当前节点和其两个子节点中较大的一个节点比较，如果较大的子节点 > 当前节点，则互换两节节点位置
 * @param k
 */
private void sink(int k) {
    while (2*k <= N) {
        int tempMaxIndex;
        if (2*k + 1 <= N) { // 证明有右子节点，接下来在左子节点、右子节点中寻找最大的那个索引
            tempMaxIndex = less(2*k, 2*k + 1) ? 2*k + 1 : 2*k;
        } else { // 没有右子节点
            tempMaxIndex = 2*k; // 将左子节点索引赋值给tempMaxIndex
        }

        // 当前k索引处的元素已经大于他的子节点元素，则直接退出循环
        if(!less(k, tempMaxIndex)) {
            break;
        }

        // 否则继续下沉
        exchange(k, tempMaxIndex);
        k = tempMaxIndex;
    }
}
```

### 5.2 最小优先队列

### 5.3 索引优先队列

## 六、树的进阶

### 6.1 平衡树

之前我们学习过二叉查找树，发现它的查询效率比单纯的链表和数组的查询效率要高很多，大部分情况下，确实是 这样的，但不幸的是，在最坏情况下，二叉查找树的性能还是很糟糕。

例如我们依次往二叉查找树中插入9,8,7,6,5,4,3,2,1这9个数据，那么最终构造出来的树是长得下面这个样子：

<img src="https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/image.30v2wd8ta9k0.webp" alt="image" style="zoom:67%;" />

我们会发现，如果我们要查找1这个元素，查找的效率依旧会很低。效率低的原因在于这个树并不平衡，全部是向 左边分支，如果我们有一种方法，能够不受插入数据的影响，让生成的树都像完全二叉树那样，那么即使在最坏情 况下，查找的效率依旧会很好。

#### 2-3查找树

##### 定义

一棵2-3查找树要么为空，要么满足满足下面两个要求：

1. 2- 节点

   含有一个键(及其对应的值)和两条链。

   左链接指向2-3树中的键都小于该结点，右链接指向的2-3树中的键都大 于该结点。

2. 3- 节点

   含有两个键（及其对应的值）、三条链

   左链接指向的2-3树中的键都小于该结点，中链接指向的2-3树中的键都 位于该结点的两个键之间，右链接指向的2-3树中的键都大于该结点。

   <img src="https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/image.3bo40wo7gx80.webp" alt="image" style="zoom:67%;" />

##### 查找

将二叉查找树的查找算法一般化我们就能够直接得到2-3树的查找算法。要判断一个键是否在树中，我们先将它和 根结点中的键比较。如果它和其中任意一个相等，查找命中；否则我们就根据比较的结果找到指向相应区间的连 接，并在其指向的子树中递归地继续查找。如果这个是空链接，查找未命中。

##### 插入

1. 向2-节点中插入新键

2. 向一颗只含有一个3-节点的树中插入新键

3. 向一个父节点为2-节点的3-节点中插入新键

4. 向一个父节点为3-节点的3-节点中插入新键

5. 分解根节点

   树的高度+1

##### 性质

通过对2-3树插入操作的分析，我们发现在插入的时候，2-3查找树需要做一些局部的变换来保持2-3查找树的平衡

一颗完全平衡的2-3查找树具有以下性质：

1. 任意空链接到根节点的路径长度都是相等的
2. 4-节点变换为3-节点时，树的高度不会发生变化。只有当根节点是临时的4-节点时，分解根节点时，树的高度+1
3. 2-3查找树与普通的二叉查找树最大的区别在于，普通的二叉查找树是自顶向下生长的，然而2-3查找树是自底向上生长的。

##### 实现

2-3查找树实现起来比较复杂，在某些情况插入后的平衡操作可能会使得效率降低。但是2-3查找树作为一种比较重 要的概念和思路对于我们后面要讲到的红黑树、B树和B+树非常重要。

#### 红黑树

我们前面介绍了2-3树，可以看到2-3树能保证在插入元素之后，树依然保持平衡状态，它的最坏情况下所有子结点 都是2-结点，树的高度为lgN,相比于我们普通的二叉查找树，最坏情况下树的高度为N，确实保证了最坏情况下的 时间复杂度，但是2-3树实现起来过于复杂，所以我们介绍一种2-3树思想的简单实现：红黑树。

红黑树主要是对2-3树进行编码，红黑树背后的基本思想是<font color="red">用标准的二叉查找树</font>(完全由2-结点构成)和一些<font color="red">额外的信息</font>(替换3-结点)来表示2-3树。我们将树中的链接分为两种类型：

1. <font color="red">红链接</font>

   将两个2-结点连接起来构成一个3-结点

2. <font color="red">黑链接</font>

   则是2-3树中的普通链接。

确切的说，我们将3-结点表示为由由一条左斜的红色链接(两个2-结点其中之一是另一个的左子结点)相连的两个2- 结点。这种表示法的一个优点是，我们无需修改就可以直接使用标准的二叉查找树的get方法。

<img src="https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/image.u4eet1muceo.webp" alt="image" style="zoom: 67%;" />

##### 红黑树定义

红黑树是含有红黑链接并满足下列条件的二叉查找树：

1. 红链接均为做链接
2. 没有任何一个节点同时和两条红链接相连
3. 该树时完美黑色平衡的

<img src="https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/image.asnj5669ef4.webp" alt="image" style="zoom:67%;" />

##### 红黑树节点API

因为每个结点都只会有一条指向自己的链接（从它的父结点指向它），我们可以在之前的Node结点中添加一个布 尔类型的变量color来表示链接的颜色。如果指向它的链接是红色的，那么该变量的值为true，如果链接是黑色 的，那么该变量的值为false。

<img src="https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/image.75yncu14eag0.webp" alt="image" style="zoom:67%;" />

```java
public class RedBlackTree<K, V> {
    /**
     * 键
     */
    public K key;
    /**
     * 值
     */
    private V value;
    /**
     * 左子节点
     */
    private Node left;
    /**
     * 右子节点
     */
    private Node right;
    /**
     * 其父节点指向该节点的颜色
     */
    private boolean color;


    public RedBlackTree(K key, V value, Node left, Node right, boolean color) {
        this.key = key;
        this.value = value;
        this.left = left;
        this.right = right;
        this.color = color;
    }
}
```

##### 平衡化

在对红黑树进行一些增删改查的操作后，很有可能会出现红色的右链接或者两条连续红色的链接，而这些都不满足 红黑树的定义，所以我们需要对这些情况通过旋转进行修复，让红黑树保持平衡。

**提示：**当前节点为h，它的右子节点为x

1. 左旋

   当某个节点的左子节点为黑色， 右子节点为红色，此时需要左旋

   <img src="https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/image.57lnubktt6w0.webp" alt="image" style="zoom: 67%;" />

   **左旋过程：**

   1. 让x的左子节点成为h的右子节点 

      ```java
      h.right = x.left;
      ```

   2. 让h节点成为x的左子节点

      ```java
      x.left = h;
      ```

   3. 让h的color属性成为x的color属性值

      ```java
      x.color = h.color;
      ```

   4. 让h的color属性变成red

      ```java
      h.color = true;
      ```

2. 右旋

   当某个节点的左子节点是红色，左子节点的左子节点也是红色，此时需要右旋
   
   <img src="https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/image.545lijhkm400.webp" alt="image" style="zoom:67%;" />
   
   **右旋过程：**
   
   1. 让x节点的右子节点成为h节点的左子节点
   2. 让h节点成为x节点的右子节点
   3. 让x的color变成h的color属性
   4. 让h的color属性为RED
   
   <font color="red">此时右旋结束后还是不能满足“红黑树中任意一个节点只能有一条红色链接与之相连”的要求，后面可以通过颜色的反转解决该问题。</font>

##### 向单个2-节点中插入新键

一棵只含有一个键的红黑树只含有一个2-结点。插入另一个键后，我们马上就需要将他们旋转。

1. 如果新键小于当前结点的键，我们只需要新增一个红色结点即可，新的红黑树和单个3-结点完全等价。

   <img src="https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/image.6tm30mfwzmg0.webp" alt="image" style="zoom: 67%;" />

2. 如果新键大于当前结点的键，那么新增的红色结点将会产生一条红色的右链接，此时我们需要通过左旋，把 红色右链接变成左链接，插入操作才算完成。形成的新的红黑树依然和3-结点等价，其中含有两个键，一条红 色链接

   <img src="https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/image.4brqkpnvaw00.webp" alt="image" style="zoom:67%;" />

##### 向底部2-节点中插入新键

用和二叉查找树相同的方式向一棵红黑树中插入一个新键，会在树的底部新增一个结点（可以保证有序性），唯一 区别的地方是我们会用红链接将新结点和它的父结点相连。如果它的父结点是一个2-结点，那么刚才讨论的两种方 式仍然适用。

<img src="https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/image.6ugpf4pvf7k0.webp" alt="image" style="zoom:67%;" />

##### 颜色反转

当一个节点的左子节点和右子节点的color均为RED时，此时该节点可以看为临时的`4-节点`，只需要将当前的节点的color变成RED，当前节点的左子节点、右子节点color变成BLACK即可。此过程可以看成4-节点的拆解过程，树的层级可能 +1。

<img src="https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/image.75ntft2s4z00.webp" alt="image" style="zoom:67%;" />

##### 向一个3-节点中插入新键

<font color="red">在红黑树中，从直观上看所有的节点都是2-节点，因此每插入一个节点都希望当前节点和其父节点组成一个3-节点，所以当前节点都是红色链接</font>

可以分为三种情况

1. 新键大于原树的两个键

   <img src="https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/image.10umdt42lm4w.webp" alt="image" style="zoom:67%;" />

2. 新键小于原树的两个键

   <img src="https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/image.2qlqpp7rwoq0.webp" alt="image" style="zoom:67%;" />

3. 新键介于原树两个键之间

   <img src="https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/image.3m3r2kt3i1s0.webp" alt="image" style="zoom:67%;" />

##### 根节点的颜色总是黑色

之前我们介绍结点API的时候，在结点Node对象中color属性表示的是父结点指向当前结点的连接的颜色，由于根 结点不存在父结点，所以每次插入操作后，我们都需要把根结点的颜色设置为黑色。

##### 向树底部的3-节点插入新键

假设在树的底部的一个3-结点下加入一个新的结点。前面我们所讲的3种情况都会出现。指向新结点的链接可能是 3-结点的右链接（此时我们只需要转换颜色即可），或是左链接(此时我们需要进行右旋转然后再转换)，或是中链 接(此时需要先左旋转然后再右旋转，最后转换颜色)。颜色转换会使中间结点的颜色变红，相当于将它送入了父结 点。这意味着父结点中继续插入一个新键，我们只需要使用相同的方法解决即可，直到遇到一个2-结点或者根结点 为止。

<img src="https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/image.3olgap16g7c0.webp" alt="image" style="zoom:67%;" />

<img src="https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/image.m4aluozth4g.webp" alt="image" style="zoom:67%;" />

<img src="https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/image.5c8k463g4io0.webp" alt="image" style="zoom:80%;" />

##### 红黑树API实现

##### 红黑树实现

```java
public class RedBlackTree<K extends Comparable<K>, V> {
    //根节点
    private Node root;
    //记录树中元素的个数
    private int N;
    //红色链接
    private static final boolean RED = true;
    //黑色链接
    private static final boolean BLACK = false;

    //结点类
    private class Node {
        //存储键
        public K key;
        //存储值
        private V value;
        //记录左子结点
        public Node left;
        //记录右子结点
        public Node right;
        //由其父结点指向它的链接的颜色
        public boolean color;

        public Node(K key, V value, Node left, Node right, boolean color) {
            this.key = key;
            this.value = value;
            this.left = left;
            this.right = right;
            this.color = color;
        }
    }

    //获取树中元素的个数
    public int size() {
        return N;
    }

    /**
     * 判断当前节点的父指向链接是否为红色
     *
     * @param x
     * @return
     */
    private boolean isRed(Node x) {
        if (x==null){
            return false;
        }
        return x.color==RED;
    }

    /**
     * 左旋
     * @param h
     * @return
     */
    private Node rotateLeft(Node h) {
        // 获取h节点的右子节点，表示为x节点
        Node x = h.right;
        // 让x节点的左子节点成为h节点的右子节点（左旋）
        h.right = x.left;
        // 让h节点成为x节点的左子节点
        x.left = h;
        // 让x节点的color属性等于h节点的color属性
        h.color = x.color;
        // 让h节点的color属性变成红色
        h.color = RED;
        return x;
    }

    /**
     * 右旋
     * @param h
     * @return
     */
    private Node rotateRight(Node h) {
        // 获取h节点的左子节点，命名为x节点
        Node x = h.left;
        // 让x节点的右子节点成为h节点的左子节点
        h.left = x.right;
        // 让h节点成为x节点的右子节点
        x.right = h;
        // 把h节点的color赋值给 x节点的color
        x.color = h.color;
        // 让h节点的color变成红色
        h.color = RED;
        return x;
    }

    /**
     * 颜色反转,相当于完成拆分4-节点
     * @param h
     */
    private void flipColors(Node h) {
        // 当前节点的颜色变成红色
        h.color = RED;
        // TODO: 不需要判断当前节点左子节点是否为空？右子节点是否为空？
        // 当前节点的左子节点、右子节点变成黑色
        h.left.color = BLACK;
        h.right.color = BLACK;
    }

    /**
     * 在整个树中完成插入操作
     * @param keyf
     * @param value
     */
    public void put(K key, V value) {
        root = put(root, key, value);
        // 由于颜色反转可能会将根节点的颜色反转成红色
        // 此时和根节点的颜色始终为黑色相违背
        // 因此需要重新将根节点的颜色赋值成黑色
        root.color = BLACK;
    }

    /**
     * 在指定树中完成插入操作
     * @param node
     * @param key
     * @param value
     * @return
     */
    private Node put(Node node, K key, V value) {
        // 1.判断node节点是否为空，如果为空则直接返回一个红色节点
        if (node == null) {
            N ++;
            return new Node(key, value, null, null, RED);
        }
        // 2.判断k的值和node节点的键的大小
        int compareResult = key.compareTo(node.key);
        if (compareResult < 0) { // 2.1 如果k < node.key，则继续向左走
            node.left = put(node.left, key, value);
        }
        else if (compareResult > 0) { // 2.2 如果k > node.key，则继续向右走
            node.right = put(node.right, key, value);
        } else { // 2.3 如果k = node.key，则直接替换value即可
            node.value = value;
        }
        // 3.在插入过程中会破坏树的平衡性，需要通过左旋、右旋、颜色反转的方式保证树的平衡性
        // 3.1 左旋(node节点的右子节点的颜色为RED)
        if (isRed(node.right) && !isRed(node.left)) {
            node = rotateLeft(node);
        }
        // 3.2 右旋(node节点的左子节点颜色为红色 && node节点的左子节点的左子节点的颜色为红色)
        if (isRed(node.left) && isRed(node.left.left)) {
            node = rotateRight(node);
        }
        // 3.3 颜色反转(node节点左子节点颜色为RED && node节点的右子节点颜色为RED)
        if (isRed(node.left) && isRed(node.right)) {
            flipColors(node);
        }
        return node;
    }

    /**
     * 在整个树中查询
     * @param key
     * @return
     */
    public V get(K key) {
        return get(root, key);
    }

    /**
     * 在指定树中查询
     * @param node
     * @param key
     * @return
     */
    private V get(Node node, K key) {
        // 判空
        if (node == null) {
            return null;
        }
        // 比较key和node的键的大小
        int compareResult = key.compareTo(node.key);
        if (compareResult > 0) { // 递归向右寻找
            return get(node.right, key);
        } else if (compareResult < 0) { // 递归向左寻找
            return get(node.left, key);
        } else { // 匹配到键值对
            return node.value;
        }
    }
}
```

### 6.2 B-树

前面我们已经学习过了二叉查找树、2-3树以及他的实现红黑树，在2-3树中，一个节点可以有多个key，它的实现红黑树中使用了对链接染色的方式表达多个key。本章节将要介绍另外一种树型结构B树，这种数据结构中，一个节点允许多于两个key的存在。

#### B树的特性

B树中允许一个结点中包含多个key，现在我们选 择一个参数M，来构造一个B树，我们可以把它称作是<font color="red">M阶的B树</font>，那么该树会具有如下特点：

1. 每个节点最多有M - 1个key，并且以升序排序
2. 每个节点最多能有M个子节点
3. 根节点至少有两个子节点

<font color="red">在实际的应用场景中，B树的阶数一般比较大（通常大于100），即便保存了大量的数据，B树的高度还是很小，还是可以保证非常高的查询效率。</font>

#### B树存储数据

#### B树在磁盘文件中的应用

### 6.3 B+树

#### B+树存储数据

#### B+树和B树的对比

#### B+树在数据库中的应用

##### 未建立主键索引查询

##### 建立主键索引查询

##### 区间查询





<font color="red"></font>
