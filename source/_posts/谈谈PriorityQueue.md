---
title: 谈谈PriorityQueue
date: 2020-10-29 15:12:52
tags:
- 数据结构
- JAVA
categories:
- 数据结构
---

## 概念

我们需要清楚的一个概念，**什么是队列**

百度百科：

队列是一种特殊的[线性表](https://baike.baidu.com/item/线性表/3228081)，特殊之处在于它只允许在表的前端（front）进行删除操作，而在表的后端（rear）进行插入操作，和栈一样，队列是一种操作受限制的线性表。进行插入操作的端称为队尾，进行删除操作的端称为队头。

换成一句话就是，是一个能够元素先进先出的数据结构。

一般的队列并不会对存储的元素下手，只会按照他们的先后入队顺序存储他们的位置。

清楚了这个，我们再来说一下PriorityQueue这个队列。

标准定义：

PriorityQueue类在Java1.5中引入。PriorityQueue是基于优先堆的一个无界队列，这个优先队列中的元素可以默认自然排序或者通过提供的Comparator（比较器）在队列实例化的时排序。要求使用Java Comparable和Comparator接口给对象排序，并且在排序时会按照优先级处理其中的元素。

PriorityQueue是一个优先级队列，也就是说他会按照一定的规律将内部存储的数据进行排序，我们可以提供Comparator来定义我们的排序形式。

举个例子：

现在该队列里有1 3 5，现在往里面插入 2，该队列的元素顺序会变成1 2 3 5。

**会自动排序的队列！**



## 底层实现



底层就是一颗二叉堆。

**大顶堆：arr[i] >= arr[2i+1] && arr[i] >= arr[2i+2]**  

**小顶堆：arr[i] <= arr[2i+1] && arr[i] <= arr[2i+2]**  

二叉堆具备以下特点：

1. 二叉堆是一个完全二叉树。
2. 根节点总是大于左右节点（大顶堆），或者小于左右节点（小顶堆）。

我们向堆内插入数据是自低向上插入

我来模拟一下排序过程：

先是随便整个完全二叉树

![大顶堆](https://gitee.com/lehanbal/blog-image/raw/master/img/大顶堆.png)

好！他很乱！

从底部开始！

![大顶堆的子节点](https://gitee.com/lehanbal/blog-image/raw/master/img/大顶堆的子节点.png)

给他按照相关规则排序！

![大顶堆排序1](https://gitee.com/lehanbal/blog-image/raw/master/img/大顶堆排序1.png)

好！该节点完事，下一个节点！

![大顶堆的子节点1](https://gitee.com/lehanbal/blog-image/raw/master/img/大顶堆的子节点1.png)

继续排序！

![大顶堆排序2](https://gitee.com/lehanbal/blog-image/raw/master/img/大顶堆排序2.png)

排序完毕。

但是这只是一个比较凑巧的情况，如果顶部的68被换到左子树上，并且比左子树上的左右子树还要小的话怎么办呢？

~~因为例子没有出现所以不讨论~~

这时候就需要对这个子堆再进行一次堆排序，也就是说，如果我们对堆顶的元素位置改变了，那么我们就需要递归的去对他的子堆再来一次排序。后续我会专门说一次堆排序，这里就大致提一下。

我刚刚模拟了一边堆排序的过程，在PriorityQueue队列中，每加入一个元素就相当于在刚刚那样的完全二叉树底部插入一个元素并且再进行一次刚刚的堆排序模拟。

底层数据结构看了，我们看看源码。

## 源码

**属性**

```java
//默认初始容量
private static final int DEFAULT_INITIAL_CAPACITY = 11;
//创建的一个队列，队列内的元素下标满足 queue[i]的子节点为queue[2*i+1]和queue[2*i+2]
transient Object[] queue;
//队列内的元素个数
int size;
//比较器，用于升序或者降序
private final Comparator<? super E> comparator;
//队列修改次数
transient int modCount;
```

**构造方法**

`PriorityQueue()`  ：创建一个PriorityQueue ，具有默认的初始容量（11），根据它们的自然顺序对其元素进行排序 。 

`PriorityQueue(Collection<? extends E> c)`  ：创建一个collection集合中的元素的PriorityQueue。 

`PriorityQueue(Comparator<? super E> comparator)`  ：创建具有默认初始容量的PriorityQueue，并根据指定的比较器对其元素进行排序。 

`PriorityQueue(int initialCapacity)` ：使用指定的初始容量创建一个 PriorityQueue，并根据其自然顺序对元素进行排序。

`PriorityQueue(PriorityQueue<? extends E> c)`：  创建包含 `PriorityQueue`优先级队列中的元素的PriorityQueue。 

`PriorityQueue(SortedSet<? extends E> c)`  ：创建一个包含指定排序set集中的元素的PriorityQueue。 

**方法**

**（1）add：插入一个元素，不成功会抛出异常**

调用的offer方法。

**（2）offer：插入一个元素，不能被立即执行的情况下会返回一个特殊的值（true 或者 false）**

```java
    public boolean offer(E e) {
        if (e == null)
            throw new NullPointerException();
        modCount++;//修改次数加一
        int i = size;
        if (i >= queue.length)
            grow(i + 1);
        siftUp(i, e);
        size = i + 1;
        return true;
    }
```

插入空元素是会报空指针异常的。会先判断队列容量够不够，不够就调用grow来进行队列的扩容，然后插入操作就进入了siftUp()函数

```java
    private void siftUp(int k, E x) {
        if (comparator != null)
            siftUpUsingComparator(k, x, queue, comparator);
        else
            siftUpComparable(k, x, queue);
    }
```

这里判断我们是否传入了比较器，之后便是插入操作。

```java
    private static <T> void siftUpComparable(int k, T x, Object[] es) {
        Comparable<? super T> key = (Comparable<? super T>) x;
        while (k > 0) {
            int parent = (k - 1) >>> 1;
            Object e = es[parent];
            if (key.compareTo((T) e) >= 0)
                break;
            es[k] = e;
            k = parent;
        }
        es[k] = key;
    }
```

**（3）remove：删除一个元素，如果不成功会返回false。**

调用的其实是removeAt()

```java
    public boolean remove(Object o) {
        int i = indexOf(o);
        if (i == -1)
            return false;
        else {
            removeAt(i);
            return true;
        }
    }
```

```java
    E removeAt(int i) {
        // assert i >= 0 && i < size;
        final Object[] es = queue;
        modCount++;
        int s = --size;
        if (s == i) // removed last element
            es[i] = null;
        else {
            E moved = (E) es[s];
            es[s] = null;
            siftDown(i, moved);
            if (es[i] == moved) {
                siftUp(i, moved);
                if (es[i] != moved)
                    return moved;
            }
        }
        return null;
    }
```

堆排序的上浮与下沉。

```java
    private void siftDown(int k, E x) {
        if (comparator != null)
            siftDownUsingComparator(k, x, queue, size, comparator);
        else
            siftDownComparable(k, x, queue, size);
    }
```

判断比较器

```java
    private static <T> void siftDownComparable(int k, T x, Object[] es, int n) {
        // assert n > 0;
        Comparable<? super T> key = (Comparable<? super T>)x;
        int half = n >>> 1;           // loop while a non-leaf
        while (k < half) {
            int child = (k << 1) + 1; // assume left child is least
            Object c = es[child];
            int right = child + 1;
            if (right < n &&
                ((Comparable<? super T>) c).compareTo((T) es[right]) > 0)
                c = es[child = right];
            if (key.compareTo((T) c) <= 0)
                break;
            es[k] = c;
            k = child;
        }
        es[k] = key;
    }
```

**（4）poll：删除一个元素，并返回删除的元素**

调用的siftDown()函数。

```java
    public E poll() {
        final Object[] es;
        final E result;

        if ((result = (E) ((es = queue)[0])) != null) {
            modCount++;
            final int n;
            final E x = (E) es[(n = --size)];
            es[n] = null;
            if (n > 0) {
                final Comparator<? super E> cmp;
                if ((cmp = comparator) == null)
                    siftDownComparable(0, x, es, n);
                else
                    siftDownUsingComparator(0, x, es, n, cmp);
            }
        }
        return result;
    }
```

**（5）peek：查询队顶元素**

```java
    public E peek() {
        return (E) queue[0];
    }
```

**（6）indexOf(Object o)：查询对象o的索引**

```java
    private int indexOf(Object o) {
        if (o != null) {
            final Object[] es = queue;
            for (int i = 0, n = size; i < n; i++)
                if (o.equals(es[i]))
                    return i;
        }
        return -1;
    }
```

**（7）contains(Object o)：判断是否容纳了元素**

```java
    public boolean contains(Object o) {
        return indexOf(o) >= 0;
    }
```

