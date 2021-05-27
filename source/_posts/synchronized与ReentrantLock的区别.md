---
title: synchronized与ReentrantLock的区别
date: 2020-11-20 23:14:55
tags:
- JAVA
- JUC
categories:
- JAVA
---

[原文章](https://blog.csdn.net/zxd8080666/article/details/83214089)，本来想自己总结以下的，但是这文章我觉得写得太好了，自己总结的内容完全没有这个文章写得好，直接搬过来了。

## 相同点

`synchronized`与`ReentrankLock`都是通过加锁实现同步，并且都是阻塞式同步，也就是锁如果一个线程获得了对象锁，在该线程进行访问该同步代码块的时候，其他线程必须阻塞等待该线程访问完毕。

进行线程阻塞和唤醒的代价是比较高的，操作系统需要在用户态和内核态之间来回切换。

## 功能区别

`synchronized`是Java的关键字，原生语法，通过JVM实现。

`ReentrantLock`是JDK 1.5之后提供的API层面的互斥锁，需要`lock()`和`unlock()`方法配合`try..finally`语句块实现。

便利性：`synchronized`的使用很便捷，我们并不需要手动去释放锁，这一切都交给编译器去操作，并且在wait的过程中，会解锁。`ReentrantLock`则需要手动进行加锁和解锁，若忽略解锁步骤就会造成死锁，如果对`ReentrantLock`进行wait()方法的调用，该锁依旧不会被释放，抱着锁睡觉。

锁的细粒度和灵活度：`ReentrantLock`锁的细粒度和灵活度都优于`synchronized`。

## 性能区别

在`synchronized`优化之前，两者的性能差很多。

但是`synchronized`引入了偏向锁，自旋锁之后，两者性能差不多持平，在两者方法都可用的情况下，官方推荐使用`synchronized`。

### `synchronized`

`synchronized`经过编译，会在同步代码块的前后分别形成`monitorenter`和`monitorexit`两个字节码指令。在执行`monitorenter`指令的时候，首先要尝试获取对象锁。如果这个对象没被锁定，或者当前线程已经拥有该对象锁，就把锁的计算器加一，相应的，在执行`monitorexit`的时候计算器减一。当计算器为零的时候，锁就被释放了。如果获取当前对象锁失败，那么当前线程就要阻塞，直到对象锁被另一个线程释放为止。

### `ReentrantLock`

由于`ReentrantLock`是`java.util.concurrent`包下提供的一套互斥锁，相比`Synchronized`，`ReentrantLock`类提供了一些高级功能，主要有以下3项：

1. 等待可中断，持有锁的线程长期不释放的时候，正在等待的线程可以选择放弃等待，这相当于Synchronized来说可以避免出现死锁的情况。通过`lock.lockInterruptibly()`来实现这个机制。
2. 公平锁，多个线程等待同一个锁时，必须按照申请锁的时间顺序获得锁，`Synchronized`锁非公平锁，`ReentrantLock`默认的构造函数是创建的非公平锁，可以通过参数true设为公平锁，但公平锁表现的性能不是很好。
3. 锁绑定多个条件，一个`ReentrantLock`对象可以同时绑定对个对象。`ReenTrantLock`提供了一个`Condition`类，用来实现分组唤醒需要唤醒的线程们，而不是像`synchronized`要么随机唤醒一个线程要么唤醒全部线程。

`ReenTrantLock`的实现是一种自旋锁，通过循环调用CAS操作来实现加锁。