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

##### 中间值问题

##### 单项链表是否有环问题

##### 有环链表入口问题

#### 1.2.6 循环链表

#### 1.2.7 约瑟夫问题

