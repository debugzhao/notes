## 一、线性表

### 1.1 顺序表

### 1.2 链表

#### 1.2.1 单向链表

#### 1.2.2 双向链表

#### 1.2.3 链表的复杂度分析

#### 1.2.4 链表反转

##### 递归算法

链表反转内部是用[递归算法](https://cloud.tencent.com/developer/article/1356049)解决的。递归算法主要分为两个阶段，第一阶段：递归分解任务；第二阶段：回归分治任务

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

### 5.2 最小优先队列

### 5.3 索引优先队列





<font color="red"></font>
