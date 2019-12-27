---
title: "Union-Find Algorithm, 并查算法"
excerpt: "包括快速查找，快速合并，算法的时间复杂度改进"
date: 2019-12-25
categories: [Algorithm]
tags: [Algorithm, Java]
---

这是[Algorithm, Part I](https://www.coursera.org/learn/algorithms-part1/)的第一周内容——并查算法的学习笔记，观看之后特来写一篇博客以加深理解和记忆。这也是我的GitHub Pages的第一篇博客。


## 目录

1. 动态连通性
2. 快速查询
3. 快速合并
4. 改进算法：加权快速查询
5. 改进算法：快速查询的路径压缩方法

## 1. 动态连通性

### 1.1 并、查操作简介

- 合并(Union): 将两个对象连接在一起
- 查询(Find): 查询两个对象是否相连，即两个对象之间是否存在一条路径

现在有7个对象，分别是数字0到6，对它们进行如下操作：

union(0, 2),
union(1, 3),
union(1, 4),
union(5, 6),
find(1, 4),
find(4, 5),
find(5, 6)

![union-to-get-three-component](/assets/images/algorithm/2019-12-25-union-find/union-to-get-three-component.png)

以上的四个Union操作分别连通{0, 2}, {1, 3, 4}, {5, 6}.

find(1, 4)查询1和4是否连通，结果返回是，后续find操作类似。

### 1.2 连通分量

连通分量：是存在连接的对象的一个最大的集合

上面的Union操作就产生了3个连通分量：{0,2}, {1,3,4}, {5,6}

现在我们将2和5连接起来：union(2, 5)

![union-2-5](/assets/images/algorithm/2019-12-25-union-find/union-2-5.png)

现在只剩下2个连通分量：{0,2,5,6}, {1,3,4}

### 1.3 连通性相关的应用

- 网络世界中各个电脑的连接

  可以查询网络中的某两台主机是否存在连接，是否需要建立一条连接使得它们能够交换数据。

- 社交网络中的朋友关系

  你和你的朋友存在朋友关系，你可以通过你的朋友联系上你朋友的朋友，这是存在连通性；而你与你朋友的朋友建立友谊时，这就是union操作。

- 电路中的元件连接性

  两个元件之间是否存在电路连接，是串联还是并联？

## 2. 快速查询(Quick Find)

### 2.1 数据结构

假设存在N个对象，那么定义一个长度为N的数组id，索引号即是对象的序列号。如果索引p和q的值id[p], id[q]一样，则对象p和q存在一条路径使它们相连接。

### 2.2 查询操作

只需要检查p和q的值是否一样

### 2.3 合并操作

对对象p和q进行合并，将所有值为id[p]的索引改为id[q]
![union-in-quick-find](/assets/images/algorithm/2019-12-25-union-find/union-in-quick-find.png)

### 2.4 Quick-Find类实现

```java
public class QuickFindUF
{
  private int id[];

  public void QuickFindUF(int N)
  {
    id = new int[N];
    for (int i=0; i<N; i++)
      {
        //这里表示每个对象都不存在连接
        id[i] = i;
      }
  }

  public boolean find(int p, int q)
  {
    //这里要注意p和q都在数组长度以内，否则会导致数组溢出错误

    return id[p] == id[q]
  }

  public void union(int p, int q)
  {
    int pid = id[p];
    int qid = id[q];

    for(int i=0; i<id.length; i++)
    {
      if(id[i] == pid)
        id[i] = qid;
    }
  }
}
```

#### 时间复杂度分析

| Algorithm | initialize | find | union
| --- | ---: | ---: | ---: |
| Quick-Find | N | 1 | N |

- 查询操作的时间复杂度为O(1)，因为函数之中只有一个if语句
- 而合并操作的时间复杂度为O(n), 因为每次合并都需要遍历一次数组，如果对n个对象进行合并，那么数组的访问次数则为N²，这很低效。

## 3. 快速合并(Quick Union)

在这里我们使用树来表示连通性，一棵树中的所有节点是连通的

### 3.1 数据结构

我们仍然需要一个数组id，但与快速查询不同的是，快速合并的id数组保存的是该索引所在的树的父节点。
![data-structure-of-quick-union](/assets/images/algorithm/2019-12-25-union-find/data-structure-of-quick-union.png)

### 3.2 合并操作

对对象p和q进行合并，将p的根节点设置为q的根节点的子节点。
![union-of-quick-union](/assets/images/algorithm/2019-12-25-union-find/union-of-quick-union.png)

### 3.3 查询操作

若要检查p和q的连通性，只需检查它们的根节点是否相同

### 3.4 Quick-Union类实现

```java
public class QuickUnionUF
{
  private int id[];

  //辅助方法：查询某个节点的根节点
  private int rootOf(int x)
  {
    while(x != id[x])
      x = id[x];
    return x;
  }

  //初始化方法相同
  public void QuickUnionUF(int N)
  {
    id = new int[N];
    for (int i=0; i<N; i++)
    {
      id[i] = i;
    }
  }

  public boolean find(int p, int q)
  {
    return rootOf(p) == rootOf(q);
  }

  public void union(int p, int q)
  {
    int rootOfP = rootOf(p);
    //将p的根节点的值设置为q的根节点，即将p的根节点设置为q的根节点的子节点。
    id[rootOfP] = rootOf(q);
  }
}
```

#### 时间复杂度分析

| Algorithm | initialize | find | union
| --- | ---: | ---: | ---: |
| Quick-Union | N | Tree height | Tree height |

最坏情况下，合并之后所有的对象都在一棵树内，并且除了叶子节点以外，每个节点都只有一个子节点，即树的高度为N。

这样的话，rootOf函数的时间复杂度就为O(N)，那么find和union函数的时间复杂度也为O(N)。


## 4. 算法该如何改进？

### 4.1 改进方法1：加权

 加权并查算法是基于Quick-Union算法之上进行的改进，在进行union操作时，检查即将合并的两棵树的大小，将较小的树合并到较大的树之上。因为如果将较大的树合并到较小的树上，会造成树的高度增加，导致寻找的效率降低。

 普通Quick-Union算法可能得到的树：
![compare-quick-union](/assets/images/algorithm/2019-12-25-union-find/compare-quick-union.png)

 Weighted Quick-Union算法可能得到的树：
![compare-weighted-quick-union](/assets/images/algorithm/2019-12-25-union-find/compare-weighted-quick-union.png)

#### Weighted Quick Union类实现

```java
public class WeightedQuickUnionUF
{
  private int id[];
  //请注意，这里的sz数组存储的是以该索引作为根节点的子树的大小
  private int sz[];

  //辅助方法：查询某个节点的根节点
  private int rootOf(int x)
  {
    //检查x是否合法

    while(x != id[x])
      x = id[x];
    return x;
  }

  //初始化方法相同
  public void WeightedQuickUnionUF(int N)
  {
    id = new int[N];
    sz = new int[N];
    for (int i=0; i<N; i++)
    {
      id[i] = i;
      sz[i] = 1;
    }
  }

  public boolean find(int p, int q)
  {
    //需要检查p和q是否合法

    return rootOf(p) == rootOf(q);
  }

  public void union(int p, int q)
  {
    //检查p和q是否合法

    int rootOfP = rootOf(p);
    int rootOfq = rootOf(q);

    if(sz[rootOfp] < sz[rootOfq])
    {
      id[rootOfp] = rootOfq;
      sz[rootOfq] += sz[rootOfp];
    }
    else
    {
      id[rootOfq] = rootOfp;
      sz[rootOfp] += rootOfq;
    }
  }
}
```

#### 时间复杂度分析

| Algorithm | initialize | find | union
| --- | ---: | ---: | ---: |
| Weighted Quick-Union | N | lgN | lgN |

*其中lgN表示以2为底的对数*

通过使用加权的方法，将查询和合并的时间复杂度都变成了O(lgN)，因为在加权的情况下，树的最大高度为lgN

### 4.2 改进方法2：路径压缩

在查询对象p的根节点之后，将p设置为p的根节点的子节点，这就是Quick-Union的路径压缩方法。

![get-root-WQUPC](/assets/images/algorithm/2019-12-25-union-find/get-root-WQUPC.png)

![path-compression-exec](/assets/images/algorithm/2019-12-25-union-find/path-compression-exec.png)

#### Weighted Quick Union with Path Compression类实现

```java
public class WeightedQuickUnionPathCompressionUF
{
  private int id[];
  //请注意，这里的sz数组存储的是以该索引作为根节点的子树的大小
  private int sz[];

  //辅助方法：查询某个节点的根节点
  //路径压缩就在这个方法之中进行
  private int rootOf(int x)
  {
    //检查x是否合法

    int root = x;

    //得到根节点的索引root
    while(root != id[root])
      root = id[root];

    while(x != root)
    {
      int fatherX = id[x];
      id[x] = root; //将x设置为根节点的子节点
      x = fatherX; //处理父节点
    }
    return x;
  }

  //初始化方法相同
  public void WeightedQuickUnionPathCompressionUF(int N)
  {
    id = new int[N];
    sz = new int[N];
    for (int i=0; i<N; i++)
    {
      id[i] = i;
      sz[i] = 1;
    }
  }

  public boolean find(int p, int q)
  {
    //需要检查p和q是否合法

    return rootOf(p) == rootOf(q);
  }

  public void union(int p, int q)
  {
    //检查p和q是否合法

    int rootOfP = rootOf(p);
    int rootOfq = rootOf(q);

    if(sz[rootOfp] < sz[rootOfq])
    {
      id[rootOfp] = rootOfq;
      sz[rootOfq] += sz[rootOfp];
    }
    else
    {
      id[rootOfq] = rootOfp;
      sz[rootOfp] += rootOfq;
    }
  }
}
```

#### 时间复杂度分析

| Algorithm | initialize | find | union
| --- | ---: | ---: | ---: |
| Weighted Quick-Union | N | lg* N | lg* N |

lg* N(Iterated logarithm - 迭对数)，详情参见[Wikipedia-Iterated logarithm](https://en.wikipedia.org/wiki/Iterated_logarithm)

| N | lg* N |
| ---: | ---: |
| (-∞, 1] | 0 |
| (1, 2] | 1 |
| (2, 4] | 2 |
| (4, 16] | 3 |
| (16, 65536] | 4 |
| (65536, 2^65536] | 5 |

因此，在数据规模非常大的时候，对Quick-Union的路径压缩算法的时间复杂度也非常接近于O(1)，是最优的算法。

## 总的时间复杂度对比

| Algorithm | initialize | find | union
| --- | ---: | ---: | ---: |
| Quick-Find | N | 1 | N |
| Quick-Union | N | Tree height | Tree height |
| Weighted Quick-Union | N | lgN | lgN |
| Weighted Quick-Union | N | lg* N | lg* N |
