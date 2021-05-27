---
title: synchronized初解
date: 2020-11-19 23:26:54
tags:
- JAVA
- JUC
- synchronized
categories:
- JAVA
---

synchronized是java中的一个实现同步的关键字。

在并发编程中存在线程安全问题，多个线程共同操作共享数据就会造成数据紊乱的情况。

关键字synchronized可以保证同一时刻只能由一个线程可以执行某个方法或者某个代码块，同时synchronized可以保证一个线程的可见性（与volatile一样）。

synchronized可以修饰的对象有以下四种：

1. 修饰一个代码块，被修饰的代码块称为同步语句块，其作用范围为大括号包括起来的代码，作用的对象是调用这个代码块的对象。
2. 修饰一个方法，被修饰的方法称为同步方法，其作用范围是整个方法，作用的对象是调用这个方法的对象。
3. 修饰一个静态的方法，起作用的范围是整个静态方法，其作用的对象是这个类的所有对象。
4. 修饰一个类，其作用范围是synchronized后面括号包括的部分，作用的对象是这个类的所有对象。

为什么要同步代码块呢？在某些情况下，我们编写的方法体可能比较大，同时存在一些比较耗时的操作，而需要同步的代码又只有一小部分，如果直接对整个方法进行同步操作，可能会得不偿失，此时我们可以使用同步代码块的方式对需要同步的代码进行包裹，这样就无需对整个方法进行同步操作了。

## 多个线程同时访问synchronized修饰的方法

资源类（降低耦合度，使用了函数式接口）。

```java
package top.lehanbal.synchronizedStudy;

import java.util.concurrent.TimeUnit;

public class Resource {
    private int num = 10;

    public synchronized void ticket(){
        if(num > 0) {
            System.out.println(Thread.currentThread().getName() + "正在卖出第" + (num--) + "张票，剩余" + num + "张票");
            try {
                TimeUnit.MILLISECONDS.sleep(50);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}

public class SynchronizedDemo01 {
    public static void main(String[] args) {
        Resource resource = new Resource();
        new Thread(()->{ for (int i = 0; i < 100; i++) resource.ticket(); }, "A").start();
        new Thread(()->{ for (int i = 0; i < 100; i++) resource.ticket(); }, "B").start();
        new Thread(()->{ for (int i = 0; i < 100; i++) resource.ticket(); }, "C").start();
    }
}
```

结果如下：

```bash
A正在卖出第10张票，剩余9张票
A正在卖出第9张票，剩余8张票
A正在卖出第8张票，剩余7张票
A正在卖出第7张票，剩余6张票
A正在卖出第6张票，剩余5张票
C正在卖出第5张票，剩余4张票
C正在卖出第4张票，剩余3张票
C正在卖出第3张票，剩余2张票
B正在卖出第2张票，剩余1张票
B正在卖出第1张票，剩余0张票
```

保证了同一时刻只能有一个线程访问资源类。

## synchronized锁的对象分析

### 锁住对象的情况

在开头的修饰对象中点名到修饰一个方法的时候，作用的对象是调用这个方法的对象。

换句简单点的话来说，就是我需要访问同一个对象的其他同步方法的时候，是需要等待synchronized将锁释放掉才能够访问。

同样是资源类，两个线程访问

```java
package top.lehanbal.synchronizedStudy;

import java.util.concurrent.TimeUnit;

public class Resource {
    public synchronized void testA(){
        try {
            TimeUnit.MILLISECONDS.sleep(5000);
            System.out.println("This is A test");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    public synchronized void testB(){
        System.out.println("This is B test");
    }
}

public class SynchronizedDemo02 {
    public static void main(String[] args) {
        Resource resource = new Resource();
        new Thread(resource :: testA).start();
        new Thread(resource :: testB).start();
    }
}

```

结果永远如下，无论测试多少遍：

```bash
This is A test
This is B test
```

原因是synchronized会给调用的对象加锁，当线程所调用的对象是同一个对象的时候，就需要去等待上一个线程释放锁。

### 锁住类的情况

对于开头说的第三第四点，当synchronized修饰静态方法或者类的时候，锁的对象就会变成整个类。

直接上代码。

修饰静态代码：

```java
package top.lehanbal.synchronizedStudy;

import java.util.concurrent.TimeUnit;

public class Resource {
    public static synchronized void testA(){
        try {
            TimeUnit.MILLISECONDS.sleep(5000);
            System.out.println("This is A test");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    public static synchronized void testB(){
        System.out.println("This is B test");
    }
}

public class SynchronizedDemo03 {
    public static void main(String[] args) {
        new Thread(Resource :: testA).start();
        new Thread(Resource :: testB).start();
    }
}
```

对于使用synchronized修饰类的情况如下：

```java
package top.lehanbal.synchronizedStudy;

import java.util.concurrent.TimeUnit;

public class Resource {
    public void testA(){
        synchronized (Resource.class){
            try {
                TimeUnit.SECONDS.sleep(2);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + "-> A");
        }
    }

    public void testB(){
        synchronized (Resource.class){
            System.out.println(Thread.currentThread().getName() + "-> B");
        }
    }
}

public class SynchronizedDemo04 {
    public static void main(String[] args) {
        Resource resource1 = new Resource();
        Resource resource2 = new Resource();
        new Thread(resource1 :: testA).start();
        new Thread(resource2 :: testB).start();
    }
}
```

锁住的是整个类，所以需要等类的锁释放掉了才能够让下一个线程执行对应的代码。

```bash
Thread-0-> A
Thread-1-> B
```

