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
