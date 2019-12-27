---
title: "Analysis of Algorithms, 算法分析"
excerpt: "使用科学方法分析算法的性能"
category: [Algorithm]
comments: true
toc: true
date: 2019-12-25
last_modified_at: 2019-12-27
---

这仍然是[Algorithm, Part I](https://www.coursera.org/learn/algorithms-part1/)的第一周内容——算法分析的学习笔记，这次的内容对我来说可能有一点难度，所以需要更多时间来学习和理解。特别是其中的上界和下界的分析，后面我会单独发一篇笔记来记录对上下界的理解。

## 目录
1. 算法分析
2. 观察算法的运行时间
3. 数学模型
4. 增长阶数的分类
5. 算法理论
6. 内存分析


## 1. 算法分析

当我们在电脑上处理比较困难的问题，或者处理大型数据时，我们一般都会考虑以下的两个有关于性能的问题：

1. 这个程序的处理过程需要花费多少时间？
2. 这个程序的运行需要耗费多少内存？

举个例子：我们在某个自然语言处理的项目之中使用大型的语料库，用Python对其进行预处理时，预处理会进行多久？而我们看看电脑的任务管理器，运行该程序占用了电脑的多少内存？会不会使我们在进行其他任务时内存不足从而造成卡顿？这些都是我们需要考虑的问题。

为了解决这样的疑惑，我们可以使用 **科学方法(Scientific Method)** 对问题进行分析和建模。

科学方法分析如下：
1. Observe：**观察** 并且精确测量自然世界之中的一些数据和特性
2. Hypothesize：假设一个与观察结果一致的 **模型**
3. Predict：通过模型对未出现的数据进行 **预测**
4. Verify：通过进一步观察来 **验证** 我们的预测
5. Validate：重复证实，直到假设和观察结果一致为止


## 2. 观察算法的运行时间

我们先来看一下3-Sum算法，对于N个数，3-Sum算法计算3个整数加起来为0的三元组的组数，具体代码如下：

```java
public static int threeSum(int[] a)
{ // Count triples that sum to 0.
  int N = a.length;
  int cnt = 0;
  for (int i = 0; i < N; i++)
    for (int j = i+1; j < N; j++)
      for (int k = j+1; k < N; k++)
        if (a[i] + a[j] + a[k] == 0)
          cnt++; //和为0的三元组出现的次数
  return cnt;
}
```

对于输入数据的规模，使用Stopwatch类计算出3-Sum算法的运行时间（以下是近似时间）：

| Size of int | elapsed time(seconds) |
| ---: | ---: |
| 8 | 0.0 |
| 1000 | 0.4 |
| 2000 | 3.7 |
| 4000 | 30.0 |

————————————————————————————分割线——————————————————————————————

对于以下的输入规模和运行时间：

| N | time(second) |
| ---: | ---: |
| 1K | 0.1 |
| 2K | 0.8 |
| 4K | 6.4 |
| 8K | 51.1 |

我们使用标准坐标系对数据进行分析：

*T(N)是问题规模为N的情况下的运行时间。*

![standard-plot](/assets/images/algorithm/2019-12-26-analysis-of-algorithms/standard-plot.png)

坐标系的横纵坐标都取对数的情况下，称为log-log plot，此时的拟合的曲线为一次方程：

![log-log-plot](/assets/images/algorithm/2019-12-26-analysis-of-algorithms/log-log-plot.png)

那么我们有方程log(T(N)) = b*logN + c = a * N^b，其中a = 2^c

其中b = 2.999, c = -33.2103，有了这个假设，我们可以对16K规模的运行时间进行预测，并判断观察的结果与假设是否一致。


## 3. 数学模型

观察以下2-Sum代码：

```java
public static int twoSum(int[] a)
{
    int N = a.length;
    int count = 0;
    for(int i=0; i<N-1; i++)
        for(int j=i+1; j<N; j++)
            if(a[i] + a[j] == 0)
                count++;

    return count;
}
```

请问两个for循环运行了if语句多少次？

答案是1/2N(N-1)次

那么对于算法的运行时间我们需不需要这么准确地去计算类似for循环的循环次数呢？也不需要，我们只需要对他们进行粗略的估计即可。使用波浪符号~(Tilde Notation)进行时间估算。

那么1/2N(N-1) ~ 1/2N²

同时，我们可以忽略低阶的项，只取最高阶的项。

下图是对3-Sum算法进行的时间估算：

![ignore-lower-order](/assets/images/algorithm/2019-12-26-analysis-of-algorithms/ignore-lower-order.png)

对于近似符号~，我们有定义：

![tilde-notation](/assets/images/algorithm/2019-12-26-analysis-of-algorithms/tilde-notation.png)


## 4. 增长阶数分类

常见的增长阶数(order-of-growth): 1, logN, N, N*logN, N², N³, 2^N

![order-of-growth](/assets/images/algorithm/2019-12-26-analysis-of-algorithms/order-of-growth.png)

可以看出在大规模输入下，即使输入为512K, logN的运行时间也接近于T, 而N*logN的增长趋势和N也非常相近。

### 二叉搜索优化(Binary Search)

二叉搜索的运行时间为logN:

```java
public static int binarySearch(int[] a, int key)
{
  int left = 0;
  int right = a.length-1;

  while(left <= right)
  {
    int mid = left + (left + right) / 2;
    if(key < mid)
      right = mid - 1;
    else if(key > mid)
      left = mid + 1;
    else
      return mid;
  }
  return -1;
}
```

以上的while循环最多运行 *1 + logN* 次，证明请Google，因为我现在也说不清楚。

我们知道使用双层for循环的2-Sum算法的运行时间正比于N²，那么现在我们使用二叉搜索来优化2-Sum算法：

```java
//数组a中的元素不可存在重复，否则可能产生计算错误。
public static int fastTwoSum(int[] a)
{
    int N = a.length;
    int count = 0;
    Arrays.sort(a); //先进行排序

    for(int i=0; i<N; i++)
    {
        int j = Arrays.binarySearch(a, -a[i]); //二分查找与a[i]值相反的数
        if(j > i)
            count++;
    }
    return count;
}
```

在一个for循环内使用二分查找法，我们可以知道运行时间是正比于N*logN的。

同样的，我们使用二叉搜索优化3-Sum算法：

```java
public static int fastThreeSum(int[] a)
{
  int N = a.length;
  int count = 0;
  Arrays.sort(a);

  for(int i=0; i<N; i++)
    for(int j=i+1; j<N; j++)
    {
      int k = Arrays.binarySearch(a, -(a[i] + a[j]));
      if(k > j)
        count++;
    }
  return count;
}
```

也是相同的道理，在两个for循环内使用二分查找，那么运行时间就正比于N²*logN。

两个算法优化之后的性能对比如下：

![N-vs-logN](/assets/images/algorithm/2019-12-26-analysis-of-algorithms/N-vs-logN.png)

## 5. 算法理论

分析算法的性能，我们经常需要考虑输入的最好情况、最坏情况和平均情况。

具体分析将在[下篇博客](https://cloudzjw.github.io/)之中讲解。


## 6. 内存分析

首先，1 byte = 8 bits

然后我会对Java中的基本类型、数组和对象所占的内存进行分析。


### 6.1 基本类型

| type | bytes |
| --- | ---: |
| boolean | 1 |
| byte | 1 |
| char | 2 |
| int | 4 |
| float | 4 |
| long | 8 |
| double | 8 |

比如int类型在Java的内存分配中占32bits，那么它的值就有2^32个。


### 6.2 包含N个元素的数组

| type | bytes |
| --- | ---: |
| char[] | 2N+24 |
| int[] | 4N+24 |
| double[] | 8N+24 |

N的倍数很容易理解，一个int占4 bytes，N个int就占4N bytes，那么这24字节是用来做什么的呢？它们是用来存储数组的其他信息和填充(padding)的。

*An array of primitive-type values typeically requires 24 bytes of header information (16 bytes of object overhead, 4 bytes for the length, and 4 bytes of padding).*


### 6.3 对象

**对于一个Data类：**

```java
public class Date
{
  private int day;
  private int month;
  private int year;

  ...
}
```

分析：首先是16 bytes of overhead，然后3个int类型占用了`3*4 = 12 bytes`，最后是4 bytes of padding，总共是32 bytes。

**对于一个String类：**

```java
public class String
{
  private char[] value;
  private int offset;
  private int count;
  private int hash;

  ...
}
```

分析：16 bytes of overhead, 12 bytes for three instances of int, 4 bytes for padding, 2N+24 bytes for char[], **然后有8 bytes是对char[]数组的引用(8 bytes of reference)**，总共是2N+64 bytes。

**对于一个Counter类：**

```java
public class Counter
{
  private String name;
  private int count;

  ...
}
```

分析：16 bytes of overhead, 8 bytes of reference(String instance), 4 bytes for an instance of int, 4 bytes for padding, 如果String类占用的内存是上述我们定义的String类，那么总共就是`32 + (2N+64) bytes = 2N+96 bytes`。
