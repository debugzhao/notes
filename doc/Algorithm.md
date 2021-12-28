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

