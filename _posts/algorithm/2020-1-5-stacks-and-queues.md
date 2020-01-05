---
title: "Stacks and Queues, 栈和队列分析"
excerpt: "介绍栈和队列的实现方式以及应用"
date: 2020-01-05
category: [Algorithm]
tags: [Algorithm, Java]
---

这是[Algorithm, Part I](https://www.coursera.org/learn/algorithms-part1/)的第二周内容--栈与队列的实现以及应用。

## 目录
1. 栈
2. 队列
3. 迭代器

栈和队列是两种不同的数据结构，它们都是对象的集合，但是存储和操作的方式都不同。

![stack-and-queue](/assets/images/algorithm/2020-1-5-stacks-and-queues/stack-and-queue.png)

栈是LIFO的结构，即后进先出；而队列是FIFO的结构，即先进先出；栈可以理解为羽毛球筒，最后放进去的羽毛球是最先拿出来使用的，队列则可以理解为日常的排队，先到达的人先进行服务。

## 1. 栈

有两种实现栈的方式，一是使用链表，二是使用数组，两种实现方式各有优势。

### 链表实现

对于链表实现，每个元素都是一个节点对象，每个节点包括该节点的值以及指向下一个节点的指针。节点类的实现如下：

```java
class Node
{
  String item;
  Node next;
}
```

![linked-list-stack](/assets/images/algorithm/2020-1-5-stacks-and-queues/linked-list-stack.png)

如上图，栈的链表实现需要维护一个first节点，在这个节点进行栈的加入(push)和弹出(pop)操作。具体实现如下：

```java
public class LinkedStackOfStrings
{
  private Node first = null;
  private class Node
  {
    String item;
    Node next;
  }

  public boolean isEmpty()
  {
    return first == null;
  }

  public void push(String item)
  {
    Node oldfirst = first;
    first = new Node();
    first.item = item;
    first.next = oldfirst;
  }

  public String pop()
  {
    String item = first.item;
    first = first.next;
    return item;
  }
}
```

*由于push和pop操作中都不存在循环，所以每个操作都是耗费常数时间。*

### 数组实现

在数组使用时，我们需要定义数组的大小。具体代码如下：

```java
public class FixedCapacityStackOfStrings
{
  private String[] s;
  private int N = 0;

  public FixedCapacityStackOfStrings(int capacity)
  {
    s = new String[capacity];
  }

  public boolean isEmpty()
  {
    return N == 0;
  }

  public void push(String item)
  {
    s[N++] = item;
  }

  public String pop()
  {
    return s[--N];
  }
}
```

上面的代码使用了数组进行实现，这会出现几个问题：

1. 当数组为空时的pop操作与当数组已满时的push操作引起的溢出问题(underflow and overflow)
2. 对象游离：pop操作虽然取消了对栈顶的引用，但该对象仍存在于内存中

我们先解决第二个问题：

```java
public String pop()
{
  String popItem = s[--N];
  s[N] = null;
  return popItem;
}
```

这样Java的垃圾回收器便可以回收弹出对象的内存。

现在我们来讨论第一个问题：在客户端使用栈数据结构这个API时，我们不会去考虑栈的容量大小，因为这是API的具体实现去考虑的，所以不应该由调用栈的客户端来定义栈的具体容量。

考虑到性能问题，我们无法在每个push或者pop操作时都对栈的大小进行改变，这样做的时间复杂度为N²，我们可以在数组已满时，将数组扩大两倍；并且在数组元素数量只有容量的1/4时，将数组容量减小到一半。具体代码如下：

```java
public class ResizingArrayStackOfStrings
{
  private String[] s;
  private int N = 0;

  public ResizingArrayStackOfStrings()
  {
    //初始容量为1
    s = new String[1];
  }

  private void resize(int capacity)
  {
    String[] copy = new String[capacity];
    for(int i=0; i<N; i++)
    {
      copy[i] = s[i];
    }
    s = copy;
  }

  public boolean isEmpty()
  {
    return N == 0;
  }

  public void push(String item)
  {
    if(N == s.length) resize(s.length*2);
    s[N++] = item;
  }

  public String pop()
  {
    String popItem = s[--N];
    s[N] = null;
    if(N>0 && N == s.length/4) resize(s.length/2);
    return popItem;
  }
}
```

**时间复杂度分析**

| Operations | best | worst | amortized |
| --- | ---: | ---: | ---: |
| construct | 1 | 1 | 1 |
| push | 1 | N | 1 |
| pop | 1 | N | 1 |

其中push和pop的最坏情况是操作发生时对数组进行两倍的扩大和缩小。

### 两种实现的权衡

1. 链表实现：每个操作都是固定的常数时间，但是需要花费额外的时间和空间在链表这种形式的维护之上，比如对下一个节点的引用所花费的空间。
2. 数组实现：在没有进行数组容量的改变时，每个操作的性能都会比链表实现快，并且所需要的空间也比较小，但当容量需要改变时，需要花费N的时间。

## 2. 队列

### 链表实现

由于队列是先进先出的结构，所以在链表之中维护first和last两个节点，first进行出队，last进行入队。

```java
public class LinkedQueueOfStrings
{
  private Node first = null;
  private Node last = null;
  private class Node
  {
    String item;
    Node next;
  }

  public boolean isEmpty()
  {
    return first == null;
  }

  public void push(String item)
  {
    Node oldlast = last;
    last = new Node();
    last.item = item;
    last.next = null;

    if(isEmpty())
      first = last;
    else
      oldlast.next = last;
  }

  public String pop()
  {
    String popItem = first.item;
    first = first.next;
    if(isEmpty()) last = null;
    return popItem;
  }
}
```

## 3. 迭代器

客户端不关心栈和队列是如何实现，只需要对栈和队列进行遍历，这时候就需要迭代器。

### 链表实现

```java
public class ListIterator implements Iterator<Item>
{
  private Node curr = first;

  public boolean hasNext()
  {
    return curr != null;
  }

  public Item next()
  {
    Item item = curr.item;
    curr = curr.next;
    return item;
  }
}
```

### 数组实现

```java
public class ArrayIterator implements Iterator<Item>
{
  private int curr = 0;

  public boolean hasNext()
  {
    return curr < N;
  }

  public Item next()
  {
    return s[curr++];
  }
}
