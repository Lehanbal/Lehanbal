---
title: 谈谈Deque
date: 2020-10-29 15:09:21
tags:
- 数据结构
- Deque
- JAVA
categories:
- 数据结构
---

`ArrayDeque`是`Deque`接口的一个实现，使用了可变数组，所以没有容量上的限制。同时，`ArrayDeque`是线程不安全的，在没有外部同步的情况下，不能再多线程环境下使用。`ArrayDeque`是`Deque`的实现类，可以作为栈来使用，效率高于`Stack`；也可以作为队列来使用，效率高于`LinkedList`。需要注意的是，`ArrayDeque`不支持`null`值。

说人话，就是一个比Stack栈效率高，比LinkedList队列效率高的又能当栈又能当队列用的万金油集合，但是不支持多线程，我们来详细看看这到底是个啥，为什么说他性能比Stack和LinkedList要高。

## Deque接口

继承关系如下图：

![继承关系](http://cdn.lehanbal.top/%E7%BB%A7%E6%89%BF%E5%85%B3%E7%B3%BB.png)

学习一个类，就从它的父类进行学习，所以我们从它的继承关系来看

```java
public class ArrayDeque<E>
extends AbstractCollection<E>
implements Deque<E>, Cloneable, Serializable
```

继承了Deque接口，Deque这个接口啊，提供了插入，移除和检查元素的方法。deque是“双端队列”的缩写，也就是说，只要是集成了Deque这个接口，都能够实现双端操作。

与List不同，这个接口不支持索引访问元素。

Queue、Stack以及Deque各个接口的方法对比

| Queue Method | Stack Method | Deque Method  |                 **说明**                  |
| :----------: | :----------: | :-----------: | :---------------------------------------: |
|    add(e)    |   push(e)    |  addLast(e)   |    向队尾\栈顶插入元素，失败则抛出异常    |
|   offer(e)   |      无      | offerLast(e)  |   向队尾\栈顶插入元素，失败则返回false    |
|   remove()   |    pop()     | removeFirst() |  获取并删除队首\栈顶元素，失败则抛出异常  |
|    poll()    |      无      |  pollFirst()  |  获取并删除队首\栈顶元素，失败则返回null  |
|  element()   |    peek()    |  getFirst()   | 获取但不删除队首\栈顶元素，失败则抛出异常 |
|    peek()    |      无      |  peekFirst()  | 获取但不删除队\栈顶首元素，失败则返回null |

这个类给我们分别定义了两套获取元素和插入元素的方法，一套会抛出异常，而另一套只会返回对应的状态值。

**Stack不被支持继续使用，它继承自Vector类，Vector因为性能问题已经忌用，所以它的子类也会有类似的问题**

## ArrayDeque

*ArrayDeque*是Deque的一个实现类，底层通过数组实现，为了满足可以同时在数组两端插入或删除元素的需求，该数组还必须是循环的，即**循环数组（circular array）**，也就是说数组的任何一点都可能被看作起点或者终点。

*ArrayDeque*是非线程安全的（not thread-safe），当多个线程同时使用的时候，需要程序员手动同步；另外，**该容器不允许放入null元素**，**该容器不允许放入null元素**，**该容器不允许放入null元素**。

```java
	//存储元素的数组
	transient Object[] elements;
    //头部元素索引
    transient int head;
    //尾部要加入元素的索引
    transient int tail;
```

数组遇到插入元素是一件很苦恼的事情，巧妙地利用循环数组可以直接将元素向头部之前添加元素，并不需要大面积地移动数据。

线性数组与循环数组插入元素地区别图：**（注意头指针和尾指针的位置！头指针指向的是第一个元素，尾指针则是指向最后一个元素的后一位）**

![线性数组与循环数组](http://cdn.lehanbal.top/%E7%BA%BF%E6%80%A7%E6%95%B0%E7%BB%84%E4%B8%8E%E5%BE%AA%E7%8E%AF%E6%95%B0%E7%BB%84.png)

### 源码

我们从增删查入手，以此把每个要点的老大看了，就能彻底弄懂的源码。

#### 增

offer、add、push都是根据插入的位置的不同来分别调用addFirst或者addLast。它俩的源码如下：

addFirst：

```java
    public void addFirst(E e) {
        if (e == null)//前面说过ArrayDeque不能存储空元素，这里源码再一次证明。
            throw new NullPointerException();
        final Object[] es = elements;
        es[head = dec(head, es.length)] = e;//当前头指针指向的是第一个元素，头部指针向前减一位，由于是循环队列，所以会在dec()里面判断并且循环。
        if (head == tail)//如果头指针指到尾指针了，就进行扩容，注意是先插入，后扩容，所以插入元素的时候并不需要考虑是否能插入，此时头指针永远都是指向空的位置。
            grow(1);
    }
```

addFirst所依赖的dec：

```java
    static final int dec(int i, int modulus) {
        if (--i < 0) i = modulus - 1;//如果头指针向前移动了一位就导致数组下标越界了，那么就让这个头指针指向改数组的最后一个位置。
        return i;
    }
```

addLast：

```java
    public void addLast(E e) {
        if (e == null)
            throw new NullPointerException();
        final Object[] es = elements;
        es[tail] = e;//因为是尾指针，尾指针指向的地方为空元素，所以直接将元素插入即可，移动尾指针的步骤在inc里面执行。
        if (head == (tail = inc(tail, es.length)))//判断是否需要扩容（顺便移动了尾指针）
            grow(1);
    }
```

addLast所依赖的inc：

```java
    static final int inc(int i, int modulus) {
        if (++i >= modulus) i = 0;//尾指针移动之后如果超出了该数组的最大长度，那就让尾指针指向数组的第一个位置。
        return i;
    }
```

#### 删

与增一致，remove、poll、pop都是调用pollFirst或者pollLast方法，源码如下：

pollFirst：

```java
    public E pollFirst() {
        final Object[] es;
        final int h;
        E e = elementAt(es = elements, h = head);//获取头元素
        if (e != null) {
            es[h] = null;//将当前头元素的内容设为null
            head = inc(h, es.length);//调用inc函数让头指针先后移动以为，继续指向当前队列中的第一个元素
        }
        return e;
    }
```

pollLast：

```java
    public E pollLast() {
        final Object[] es;
        final int t;
        E e = elementAt(es = elements, t = dec(tail, es.length));//因为尾指针指向的位置是空元素，所以需要先进行dec操作将尾指针进行前移，才能获取到队列中的最后一个元素。
        if (e != null)
            es[tail = t] = null;//将当前尾指针指向的内容设值为null
        return e;
    }
```

#### 查

一样一样一样，都一样，get、peek、element都是调用的elementAt方法。

elementAt：

```java
    static final <E> E elementAt(Object[] es, int i) {
        return (E) es[i];//就这么简单,根据下标查询数组内容，所以会在数组下标的参数传递上做功夫。
    }
```

举个例子：

```java
    public E peekFirst() {
        return elementAt(elements, head);//直接将头指针的位置传递即可
    }

    public E peekLast() {
        final Object[] es;
        return elementAt(es = elements, dec(tail, es.length));//尾指针先调用dec向前移动一位才能访问到数组元素，然后根据下标查询数据即可。
    }
```

增删查就这么搞定了。

#### 扩增函数

grow函数（看个乐呵）：

```java
    private void grow(int needed) {
        // overflow-conscious code
        final int oldCapacity = elements.length;//记录下原来数组的长度
        int newCapacity;//声明用于表示新数组长度的变量
        // Double capacity if small; else grow by 50%
        int jump = (oldCapacity < 64) ? (oldCapacity + 2) : (oldCapacity >> 1);//当原数组的空间小于64的时候，每次就翻倍增则（这里的jump是扩容的量，后面还需要在原数组长度的基础上再加上jump），不然的后就按原数组长度的50%增长（>>1相当于*0.5）
        if (jump < needed
            || (newCapacity = (oldCapacity + jump)) - MAX_ARRAY_SIZE > 0)//当我们扩容的长度比我们所需要的长度要小的时候，或者是我们扩容的长度已经比我们所设定的最大数组长度还要大的时候，我们会调用newCapacity方法（下一个代码详解）
            newCapacity = newCapacity(needed, jump);
        final Object[] es = elements = Arrays.copyOf(elements, newCapacity);//我们把当前的数组用新空间拷贝一份
        if (tail < head || (tail == head && es[head] != null)) {
            //tail < head 当头指针在尾指针后面的时候
            //tail == head && es[head] != null 当头指针等于尾指针且它们指向的内容不为空
            //数组满了的情况就是头指到尾了，所以需要扩容
            int newSpace = newCapacity - oldCapacity;//新空间的大小，等价于jump
            System.arraycopy(es, head,
                             es, head + newSpace,
                             oldCapacity - head);//调用系统复制数组，将原数组的元素移动到加了新空间的位置上，同时移动了头指针的位置，与尾指针分开
            for (int i = head, to = (head += newSpace); i < to; i++)
                es[i] = null;//将新空间中的元素设置为null
        }
    }
```

newCapacity：（这个函数是用来判断边界条件以及溢出处理的）

```java
    private int newCapacity(int needed, int jump) {
        final int oldCapacity = elements.length, minCapacity;
        if ((minCapacity = oldCapacity + needed) - MAX_ARRAY_SIZE > 0) {
            //minCapacity是所需要的最小扩容空间，当最小的扩容空间比我们给定的最大数组方法还要大的时候
            if (minCapacity < 0)//这里是溢出了，直接抛出异常
                throw new IllegalStateException("Sorry, deque too big");
            return Integer.MAX_VALUE;//不然的话直接返回int的最大值
            //  private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
            //原先设定的最大数组长度比int的最大值小8，所以还能有剩余空间
        }
        if (needed > jump)//当我们所需的空间比自动扩容要大，则返回我们能够完成的最小空间
            return minCapacity;
        return (oldCapacity + jump - MAX_ARRAY_SIZE < 0)
            ? oldCapacity + jump
            : MAX_ARRAY_SIZE;//自动扩容的空间如果不比最大数组长度大则返回自动扩容的容量，否则返回指定的最大数组长度。
    }
```

## LinkedList

*LinkedList*也是Deque的一个实现类，底层是一个双向链表实现的，记录着前驱和后继两个结点。

```java
    /**
     * Pointer to first node.
     */
    transient Node<E> first;

    /**
     * Pointer to last node.
     */
    transient Node<E> last;

    private static class Node<E> {
        E item;
        Node<E> next;
        Node<E> prev;

        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }
```

该集合的元素存储结构也可以形象的表示为以下图：

![LinkedList逻辑存储图](http://cdn.lehanbal.top/LinkedList%E9%80%BB%E8%BE%91%E5%AD%98%E5%82%A8%E5%9B%BE.png)

它的继承关系如下图：

![LinkedList](http://cdn.lehanbal.top/LinkedList.png)

我们能看到*LinkedList*实现了Deque和List接口，那么*LinkedList*就具备了list和deque两个接口的属性，list能够通过下标进行索引访问，deque在前面也介绍了，能够实现双向队列（同样能够当作栈或者队列来使用）。

除此之外，*LinkedList*能够存储null元素。

那么我们来看看源码。

### 源码

#### 增

与队列和栈的增加元素方法相关的操作都与linkFirst函数和linkLast函数相关，根据插入元素的位置的不同会调用相关操作。

linkFirst：

```java
    private void linkFirst(E e) {
        final Node<E> f = first;//保存当前first结点
        final Node<E> newNode = new Node<>(null, e, f);//声明一个新的结点来作为当前的头节点
        first = newNode;
        if (f == null)//如果之前是空链表，那么就让头和尾指针指向同一个元素，因为当前元素是该链表中唯一的元素
            last = newNode;
        else
            f.prev = newNode;//如果之前不是空链表，那么就让之前的头节点的前驱结点指向当前的头节点
        size++;//链表元素+1
        modCount++;//修改次数+1
    }
```

linkLast：

```java
    void linkLast(E e) {
        final Node<E> l = last;//保存当前的last结点
        final Node<E> newNode = new Node<>(l, e, null);//声明一个新的结点来作为当前的尾结点
        last = newNode;
        if (l == null)//如果之前是空链表，那么就让头和尾指针指向同一个元素，因为当前元素是该链表中唯一的元素
            first = newNode;
        else
            l.next = newNode;//如果之前不是空链表，那么就让之前的尾结点的后继结点指向新的尾结点
        size++;
        modCount++;
    }
```

既然继承了list接口，就有够通过下标进行元素操作的方法，

```java
    public void add(int index, E element) {
        checkPositionIndex(index);//检测index是否合理，必须是链表长度内才行，不然就抛出越界异常

        if (index == size)//如果要插入的元素是链表的末尾，那么就直接调用linkLast插入
            linkLast(element);
        else
            linkBefore(element, node(index));//调用linkBefore函数插入结点
    }
```

node：在add函数中已经对index进行了合法判断，所以当前的index是在链表的合理范围内。

```java
    Node<E> node(int index) {
        //我们需要找到index位置的结点，最坏的情况只需要找n/2次便能找到，因为记录了头节点和尾结点，我们只需要计算index在该链表中心的左边还是右边即可，若是在左边，则从头节点开始查找，若是在右边，则从尾结点开始查找。
        // assert isElementIndex(index);
        if (index < (size >> 1)) {//判断index是在链表中心的左边还是右边
            Node<E> x = first;//左边就从头节点开始查找
            for (int i = 0; i < index; i++)
                x = x.next;
            return x;
        } else {
            Node<E> x = last;//若在右边则从尾结点开始查找
            for (int i = size - 1; i > index; i--)
                x = x.prev;
            return x;
        }
    }
```

linkBefore：

```java
    void linkBefore(E e, Node<E> succ) {
        //succ就是我们要插入元素的后继结点，我们需要把我们的元素安排在该结点之前
        // assert succ != null;
        final Node<E> pred = succ.prev;//记录要插入元素的前驱结点
        final Node<E> newNode = new Node<>(pred, e, succ);//声明一个新的结点，前驱后继如上所说
        succ.prev = newNode;//更新succ节点的前驱，使得我们要插入的元素成为它的前驱结点。
        if (pred == null)//如果我们要插入的元素是头节点的位置，那么就让这个新的结点成为头节点。
            first = newNode;
        else//不然的话就更新前驱节点的后继结点为我们新的结点。
            pred.next = newNode;//该节点就融入进去了
        size++;//元素数量+1
        modCount++;
    }
```

整体步骤如下图：

![LinkedList加入新元素的步骤](http://cdn.lehanbal.top/LinkedList%E5%8A%A0%E5%85%A5%E6%96%B0%E5%85%83%E7%B4%A0%E7%9A%84%E6%AD%A5%E9%AA%A4.png)

#### 删

与队列、栈相关的删除操作本质上是在调用unlinkFirst与unlinkLast

unlinkFirst：

```java
    private E unlinkFirst(Node<E> f) {
        //删除头结点元素
        // assert f == first && f != null;
        final E element = f.item;//保存头节点元素，之后要返回
        final Node<E> next = f.next;//记录下当前头结点的后继结点，要让它子承父业
        f.item = null;//让当前头节点的item以及后继节点的指向都指空，有助于GC回收
        f.next = null; // help GC
        first = next;//开始子承父业
        if (next == null)//如果成了空链表的话，那么就让头尾都指向null，成为头尾都指向null的空链表，没有指向
            last = null;
        else//如果还有元素，那就让子节点的前驱结点指向空，成为头节点（头节点没有前驱结点）
            next.prev = null;
        size--;
        modCount++;
        return element;
        //如果你疑问为什么next为空了就不需要让next的前驱指针指向空，好好读我下面这句话
        //憨批，next都是null了，null能存什么元素，null就是null，null，什么都没有了
    }
```

unlinkLast

```java
    private E unlinkLast(Node<E> l) {
        //要删除的结点是尾结点
        // assert l == last && l != null;
        final E element = l.item;//保存要删除结点的元素以便之后返回
        final Node<E> prev = l.prev;//记录下当前结点的前驱结点，我们要让这个前驱结点成为尾结点
        l.item = null;//与前面的删除头节点的时候一致，让当前的尾结点的前驱结点指向和item都标记为null，有助于GC回收
        l.prev = null; // help GC
        last = prev;//开始树立新的尾结点
        if (prev == null)//同样的，如果删除了刚刚的尾结点让链表成空了，那就让头节点也指向空，彻底抛弃指向，成为头尾都指向空的两边
            first = null;
        else//如果删了尾结点还有元素的话就让目前的尾结点的next指向为null，抛弃指向上一个尾结点的指向，这样就能成为真正的尾结点了
            prev.next = null;
        size--;
        modCount++;
        return element;
    }
```

通过下标进行元素的删除：

```java
    public E remove(int index) {
        checkElementIndex(index);//一样是对索引进行合理性判断，不合理则抛出异常
        return unlink(node(index));//node(index)找到要删除的结点，然后解除它的应用
    }

	E unlink(Node<E> x) {
        // assert x != null;
        final E element = x.item;//记录下x结点的item 前驱后继结点，都记录下来
        final Node<E> next = x.next;
        final Node<E> prev = x.prev;

        if (prev == null) {//如果前驱结点是空，说明我们要删除的结点是头结点，所以让头节点指向x的后继节点即可
            first = next;
        } else {//如果不是前驱结点的话
            prev.next = next;//就让x的前驱结点的next直接指向x的后继结点
            x.prev = null;//然后将x的前驱结点的指向设为null，不让他指向任何结点
        }

        if (next == null) {//如果后继结点是空，说明我们要删除的是尾结点，所以让尾指针直接指向x的前驱结点
            last = prev;
        } else {//不然的话，就越过x，让x的后继结点的前驱指向直接指向x的前驱，再把x的后继结点指向null
            next.prev = prev;
            x.next = null;
        }
        
        x.item = null;//将x的内容设为null，这样一来，经过上述操作，x.item x.prev x.next全都指向了null
        size--;
        modCount++;
        return element;
    }
```

删除指定内容的元素：

```java
    public boolean remove(Object o) {
        //删除指定元素的操作，就是穷举当前的链表找到值相等的元素，然后调用unlink解除这个结点的前驱和后继，
        if (o == null) {//因为LinkedList能够存储null值，null值的判断需要单独进行判断
            for (Node<E> x = first; x != null; x = x.next) {
                if (x.item == null) {
                    unlink(x);//穷举找到null之后将该结点删除
                    return true;
                }
            }
        } else {
            for (Node<E> x = first; x != null; x = x.next) {
                if (o.equals(x.item)) {//一样的穷举思路
                    unlink(x);
                    return true;
                }
            }
        }
        return false;
    }
//具体思路也用在了indexOf中
    public int indexOf(Object o) {
        int index = 0;
        if (o == null) {
            for (Node<E> x = first; x != null; x = x.next) {
                if (x.item == null)
                    return index;
                index++;
            }
        } else {
            for (Node<E> x = first; x != null; x = x.next) {
                if (o.equals(x.item))
                    return index;
                index++;
            }
        }
        return -1;
    }
```

删除过程的逻辑图：

![LinkedList断开过程](http://cdn.lehanbal.top/LinkedList%E6%96%AD%E5%BC%80%E8%BF%87%E7%A8%8B.png)

#### 查

队列、栈的头尾元素查询只需要通过对头尾结点的调用即可取得

```java
    public E peek() {
        final Node<E> f = first;
        return (f == null) ? null : f.item;
    }

    public E getFirst() {
        final Node<E> f = first;
        if (f == null)
            throw new NoSuchElementException();
        return f.item;
    }

    public E getLast() {
        final Node<E> l = last;
        if (l == null)
            throw new NoSuchElementException();
        return l.item;
    }

	//以上不解读了，一眼就看明白的东西

    public E get(int index) {
        checkElementIndex(index);
        return node(index).item;//node函数之前已经解读过了，通过对index的大小判断，在链表中心的哪一段，然后利用头尾结点就能实现n/2的查找速率
    }
```

LinkedList解读完了，我们在一起来看看ArrayDeque和LinkedList吧。

## *ArrayDeque*与*LinkedList*

我说了这么多，源码也给读了，剩下就看你们来总结这两者的不同吧。

如果你总结不出，那只能说明你没看我写的文章，懒狗。

但是本人也是懒狗，这里记录下这两者的不同，以及正面回答为什么很多情况下推荐使用ArrayDeque。

1. ArrayDeque底层通过数组实现，底层是一个循环数组，动态扩容；LinkedList则是通过一个双向链表实现。
2. ArrayDeque并不能够存储null元素，有意为之，但是LinkedList能够存储null。
3. ArrayDeque进行扩容的时候会需要比较大的性能开销，LinkedList是链表实现，并不需要扩容操作，只需要指针指向即可。

但是ArrayDeque插入头尾数据和删除数据都是妥妥的O(1)复杂度，作为队列或者栈来使用的时候，还有什么比O(1)的头部尾部操作更优秀呢？对于循环数组而言，它的随机访问依旧是O(1)，链表则是O(n)，此时唯一的缺点就是扩容造成的性能开销；相比之下LinkedList是通过链表实现的，同时他还继承了List接口，除了能够当成双向队列来使用的时候，还能够通过下表进行数据的访问以及相关操作，并且链表的优点是删除元素的时候很方便只有O(1)的时间复杂度，但是删除元素的时候你需要先找到当前元素，所以还是一个O(n)的时间过程，在当作队列或者栈来说的时候，ArrayDeque在操作的复杂度上而言，完爆LinkedList。

但是读过ArrayDeque源码之后我们知道，这玩意扩容很耗空间，而且扩容的时候性能开销也是实实在在存在的，LinkedList则是动态的使用空间大小，每次添加新的结点的时候都是原地创建结点再创建应用，并不会造成先ArrayDeque那样的扩容开销，而且它能够通过下标访问，所以在不作为队列或者栈使用的时候，是有它的用武之地。

- 在我们清楚我们的操作需要消耗多少空间的时候，优先推荐使用ArrayDeque作为队列或者栈的集合。
- 若是我们不清楚消耗的空间，并且消耗的空间会持续增大的话，可以考虑使用LinkedList来作为队列或者栈的集合。（防止ArrayDeque多次地扩容造成性能浪费）。