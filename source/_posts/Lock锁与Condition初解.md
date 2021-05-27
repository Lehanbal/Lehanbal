---
title: Lock锁与Condition初解
date: 2020-11-20 20:51:23
tags:
- JAVA
- JUC
- LOCK
categories:
- JAVA
---

`synchronized`可以实现线程之间的同步互斥，在jdk1.5之后新增加了一个`ReentrantLock`类也能够实现线程的同步问题， 并且粒度比`synchronized`要小。

Lock实现提供比使用`synchronized`方法和语句可以获得的更广泛的锁定操作。  它们允许更灵活的结构化，可能具有完全不同的属性，并且可以支持多个相关联的对象`Condition`

`ReentrantLock`实现了`Lock`接口，该接口有具体的三个实现类，分别是

1. `ReentrantLock`:可重入锁。
2. `ReentrantReadWriteLock.WriteLock`：写锁，只能有一个进程访问。
3. `ReentrantReadWriteLock.ReadLock`：读锁，可以允许多个线程同时访问。

这里对`ReentrantLock`进行进一步的说明。

## 使用`ReentrankLock`实现线程同步

`ReentrankLock`的方法中`lock`和`unlock`，分别表示上锁和解锁，我们一般将`unlock`放在try..catch..finally..的finally中，为了能够确保锁能够正确释放，避免造成死锁。

另外有一点需要注意的是，`synchronized`在线程进入阻塞状态的时候，是会将锁释放的，而`ReentrankLock`不一样，即使是线程进入阻塞也不会将锁释放。

```java
package top.lehanbal.lock;

import java.util.concurrent.locks.ReentrantLock;

public class Resource {
    private int ticket = 10;
    ReentrantLock lock = new ReentrantLock();

    public void sale(){
        lock.lock();
        try {
            while(ticket > 0){
                System.out.println("线程" + Thread.currentThread().getName() + "卖出了第" + (ticket--) + "张票，还剩下"+ ticket + "张票");
                TimeUnit.MILLISECONDS.sleep(50);
            }
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            lock.unlock();
        }
    }
}

public class LockTest {
    public static void main(String[] args) {
        Resource resource = new Resource();
        new Thread(resource :: sale, "A").start();
        new Thread(resource :: sale, "B").start();
        new Thread(resource :: sale, "C").start();
    }
}
```

结果为：

```bash
线程A卖出了第10张票，还剩下9张票
线程A卖出了第9张票，还剩下8张票
线程A卖出了第8张票，还剩下7张票
线程A卖出了第7张票，还剩下6张票
线程A卖出了第6张票，还剩下5张票
线程A卖出了第5张票，还剩下4张票
线程A卖出了第4张票，还剩下3张票
线程A卖出了第3张票，还剩下2张票
线程A卖出了第2张票，还剩下1张票
线程A卖出了第1张票，还剩下0张票
```

可以看到全都是线程A卖出去的票，线程B与C都在等待A释放锁才能执行。

一个线程在卖力的干事，另外两个线程摸鱼，这种情况能叫多线程吗，我说不能。

为了防止线程摸鱼，在Condition中的await()方法就是在线程进入休眠的时候把锁让出来，方便其他线程进行作业。

```java
package top.lehanbal.lock;

import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;

public class Resource {
    private int ticket = 10;
    ReentrantLock lock = new ReentrantLock();
    Condition condition1 = lock.newCondition();

    public void sale(){
        lock.lock();
        try {
            while(ticket > 0){
                System.out.println("线程" + Thread.currentThread().getName() + "卖出了第" + (ticket--) + "张票，还剩下"+ ticket + "张票");
                condition1.await(50, TimeUnit.MILLISECONDS);
            }
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            lock.unlock();
        }
    }
}

public class LockTest {
    public static void main(String[] args) {
        Resource resource = new Resource();
        new Thread(resource :: sale, "A").start();
        new Thread(resource :: sale, "B").start();
        new Thread(resource :: sale, "C").start();
    }
}
```

结果如下：

```bash
线程A卖出了第10张票，还剩下9张票
线程B卖出了第9张票，还剩下8张票
线程C卖出了第8张票，还剩下7张票
线程B卖出了第7张票，还剩下6张票
线程A卖出了第6张票，还剩下5张票
线程C卖出了第5张票，还剩下4张票
线程A卖出了第4张票，还剩下3张票
线程C卖出了第3张票，还剩下2张票
线程B卖出了第2张票，还剩下1张票
线程A卖出了第1张票，还剩下0张票
```



## 公平锁与非公平锁

公平锁：表示线程获取锁的顺序是按照线程加锁的顺序来进行分配的，即先到先得，先进先出的顺序。

非公平锁：一种获取锁的抢占机制，是随即拿到锁的，和公平锁不一样的是想来的不一定先拿到锁，这个方式可能造成某些线程一直拿不到锁，结果是不公平的。

源码如下，根据传递的Boolean值来区分：

```java
    public ReentrantLock(boolean fair) {
        sync = fair ? new FairSync() : new NonfairSync();
    }
```

## Condition解析

synchronized关键字可以配合Object的wait()、notify()等方法实现等待/通知模式。Condition接口提供了类似的Object的监视器方法，与Lock配合实现等待通知模式。

Object监视方法与Condition方法的区别：

| 对比项                                               | Object Monitor Methods    | Condition                                                    |
| ---------------------------------------------------- | ------------------------- | ------------------------------------------------------------ |
| 前置条件                                             | 获取对象的锁              | 调用`Lock.lock()`获得锁，调用`lock.newCondition()`获取Condition对象 |
| 调用方式                                             | 直接调用：`Object.wait()` | 直接调用：`condition.await()`                                |
| 等待队列个数                                         | 一个                      | 多个                                                         |
| 当前线程释放锁并进入等待状态                         | 支持                      | 支持                                                         |
| 当前线程释放锁并进入等待状态，在等待状态中不响应中断 | 不支持                    | 支持                                                         |
| 当前线程释放锁并进入超时等待状态                     | 支持                      | 支持                                                         |
| 当前线程释放锁并进入等待状态到将来的某个时间         | 不支持                    | 支持                                                         |
| 唤醒等待队列中的一个线程                             | 支持                      | 支持                                                         |
| 唤醒等待队列中的全部线程                             | 支持                      | 支持                                                         |

### Condition接口方法

| 方法名                            | 方法内容                                                     |
| --------------------------------- | ------------------------------------------------------------ |
| `await()`                         | 造成当前线程在接到信号或被中断之前一直处于等待状态           |
| `await(long time, TimeUnit unit)` | 造成当前线程在接到信号、被中断或到达指定等待时间之前一直处于等待状态 |
| `awaitNanos(long nanosTimeout)`   | 造成当前线程在接到信号、被中断或到达指定等待时间之前一直处于等待状态。返回值表示剩余时间，如果在`nanosTimesout`之前被唤醒，返回值 = `nanosTimeout` - 消耗时间，如果返回值小于等于0则可以认定它已经超时了。 |
| `awaitUninterruptibly()`          | 造成当前信号在接到信号之前一直处于等待状态。（该方法对中断不敏感） |
| `awaitUntil(Date deadline)`       | 造成当前线程在接到信号、被中断或到达指定最后期限之前一直处于等待状态。如果没有到指定时间就被通知，则返回true，否则表示到了指定时间，返回false |
| `signal()`                        | 唤醒一个等待线程。该线程从等待方法返回前必须获得与Condition相关的锁 |
| `signalAll()`                     | 唤醒所有等待线程。能够从等待方法返回的线程必须获得与Condition相关的锁 |

## 读写锁

`ReentrankLock`具有完全互斥排他的效果，同一时间只能够有一个线程执行lock锁后面的代码，虽然能够保证线程的安全性，但是效率不高。读写锁就是为了在特定条件提高效率而出现的。

读写锁表示两个锁：

读操作相关的锁，也叫共享锁。

写操作相关的锁，也叫排他锁。

多个读锁之间不互斥，读锁与写锁互斥，多个写锁互斥。

### `ReentrantReadWriteLock.ReadLock`读锁

多个读锁同时访问资源：

```java
package top.lehanbal.lock;

import java.util.concurrent.locks.ReentrantReadWriteLock;

public class ReadResource {
    ReentrantReadWriteLock.ReadLock readLock = new ReentrantReadWriteLock().readLock();

    public void readMethod() {
        readLock.lock();
        try {
            System.out.println(System.currentTimeMillis());
            System.out.println(Thread.currentThread().getName() + "访问了");
        } finally {
            readLock.unlock();
        }
    }
}

import java.util.concurrent.locks.ReentrantLock;

public class Resource {
    static ReentrantLock lock = new ReentrantLock(true);

    public void sale() {
        System.out.println(Thread.currentThread().getName() + "运行了");
        lock.lock();
        try {
            System.out.println(Thread.currentThread().getName() + "获得锁,是否有锁等待" + lock.hasQueuedThreads());
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            System.out.println(Thread.currentThread().getName() + "释放锁,是否有锁等待" + lock.hasQueuedThreads() + "等待数量为" + lock.getQueueLength());
            lock.unlock();
        }
    }
}
```

结果如下，几乎是在同一时间进行访问的。

```bash
1605876055602
1605876055602
1605876055602
A访问了
C访问了
B访问了
```

### `ReentrantReadWriteLock.WriteLock`写锁

多个写锁互斥：

```java
package top.lehanbal.lock;

import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.ReentrantReadWriteLock;

public class WriteResource {
    ReentrantReadWriteLock lock = new ReentrantReadWriteLock();

    public void writeMethod(){
        lock.writeLock().lock();
        try{
            System.out.println(Thread.currentThread().getName() + "获取写锁，获取时间为：" + System.currentTimeMillis());
            TimeUnit.MILLISECONDS.sleep(500);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.writeLock().unlock();
        }
    }
}

public class LockTest {
    public static void main(String[] args) {
        WriteResource writeResource = new WriteResource();
        new Thread(writeResource :: writeMethod, "A").start();
        new Thread(writeResource :: writeMethod, "B").start();
        new Thread(writeResource :: writeMethod, "C").start();
        new Thread(writeResource :: writeMethod, "D").start();
    }
}
```

停顿了500ms，防止cpu执行太快了从结果看不出来。可以看出写锁都是分别获取的。

```bash
A获取写锁，获取时间为：1605876558509
B获取写锁，获取时间为：1605876559020
C获取写锁，获取时间为：1605876559520
D获取写锁，获取时间为：1605876560021
```

