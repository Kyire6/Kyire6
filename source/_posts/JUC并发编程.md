---
title: JUC并发编程
tags:
  - 笔记
  - 技巧
categories: Java
cover: https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220106013803.png
abbrlink: eb9166f8
date: 2021-05-16 16:25:44
updated: 2021-05-16 16:25:44
---
# 多线程进阶 =>JUC并发编程

## 1、什么是JUC

![img](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210404140130.png)

JUC是java.util.concurrent的简写。在jdk官方手册中可以看到juc相关的jar包有三个。

用中文概括一下，JUC的意思就是java并发编程工具包

## 2、线程和进程

> 如果不能使用一句话说出来的技术，不扎实！

进程：一个程序，QQ.exe Music.exe 程序的集合

一个进程往往可以包含多个线程，至少包含一个！

Java默认有几个线程？ 2个 main、GC

线程：线程是程序执行中一个单一的顺序控制流程

对于Java而言：Thread、Runnable、Callable

**Java真的可以开启线程吗？** 开不了

```java
public synchronized void start() {
        /**
         * This method is not invoked for the main method thread or "system"
         * group threads created/set up by the VM. Any new functionality added
         * to this method in the future may have to also be added to the VM.
         *
         * A zero status value corresponds to state "NEW".
         */
        if (threadStatus != 0)
            throw new IllegalThreadStateException();

        /* Notify the group that this thread is about to be started
         * so that it can be added to the group's list of threads
         * and the group's unstarted count can be decremented. */
        group.add(this);

        boolean started = false;
        try {
            start0();
            started = true;
        } finally {
            try {
                if (!started) {
                    group.threadStartFailed(this);
                }
            } catch (Throwable ignore) {
                /* do nothing. If start0 threw a Throwable then
                  it will be passed up the call stack */
            }
        }
    }
	// 本地方法，底层的C++，Java无法直接操作硬件
    private native void start0();
```

### 并发、并行

并发编程：并发、并行

并发（多线程操作同一个资源）

- CPU一核，模拟出来多条线程，天下武功，唯快不破，快速交替

并行（多个人一起行走）

- CPU多核，多个线程可以同时执行；线程池

```java
package com.ouwen.demo01;

/**
 * @author IRVING
 * @create 2021-04-04 14:20
 */
public class Test {

    public static void main(String[] args) {
        // 获取CPU的核心数
        // CPU密集型，IO密集型
        System.out.println(Runtime.getRuntime().availableProcessors());
    }
}
```

并发编程的本质：**充分利用CPU的资源**

所有的公司都很看重！

### 线程有几个状态

```java
public enum State {
       	//新生
        NEW,

        //运行
        RUNNABLE,

        //阻塞
        BLOCKED,

        //等待
        WAITING,

        //超时等待
        TIMED_WAITING,

        //终止
        TERMINATED;
    }
```

### wait/sleep的区别

1. **来自不同的类**

   wait => Object

   sleep => Thread

2. **关于锁的释放**

   wait会释放锁，sleep睡觉了，抱着锁睡觉，不会释放！

3. **使用的范围是不同的**

   wait：只能在同步代码块中使用

   sleep：可以在任何地方睡

## 3、Lock锁（重点）

### 传统synchronized

```java
package com.ouwen.demo01;

/**
 * 真正的多线程开发，公司中的开发，降低耦合性
 * * 线程就是一个单独的资源类，没有任何附属的操作！
 * * 1、 属性、方法
 *
 * @author IRVING
 * @create 2021-04-04 14:42
 */

public class SaleTicketDemo01 {
    public static void main(String[] args) {
        // 并发：多线程操作同一个资源类, 把资源类丢入线程
        Ticket ticket = new Ticket();
        // @FunctionalInterface 函数式接口，jdk1.8 lambda表达式 (参数)->{ 代码 }
        new Thread(() -> {
            for (int i = 1; i < 40; i++) {
                ticket.sale();
            }
        }, "A").start();
        new Thread(() -> {
            for (int i = 1; i < 40; i++) {
                ticket.sale();
            }
        }, "B").start();
        new Thread(() -> {
            for (int i = 1; i < 40; i++) {
                ticket.sale();
            }
        }, "C").start();
    }
}

// 资源类 OOP
class Ticket {
    // 属性、方法
    private int number = 30;

    // 卖票的方式
    // synchronized 本质: 队列，锁
    public synchronized void sale() {
        if (number > 0) {
            System.out.println(Thread.currentThread().getName() + "卖出了" + (number--) + "票,剩余：" + number);
        }
    }
}
```

### Lock 接口

![image-20210404145103174](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210404145103.png)

![image-20210404145122205](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210404145122.png)

![image-20210404145431054](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210404145431.png)

公平锁：十分公平；可以先来后到

**非公平锁：十分不公平；可以插队（默认）**

```java
package com.ouwen.demo01;

import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

/**
 * @author IRVING
 * @create 2021-04-04 14:42
 */

public class SaleTicketDemo02 {
    public static void main(String[] args) {
        // 并发：多线程操作同一个资源类, 把资源类丢入线程
        Ticket2 ticket = new Ticket2();

        new Thread(() -> {
            for (int i = 1; i < 40; i++) {
                ticket.sale();
            }
        }, "A").start();
        new Thread(() -> {
            for (int i = 1; i < 40; i++) {
                ticket.sale();
            }
        }, "B").start();
        new Thread(() -> {
            for (int i = 1; i < 40; i++) {
                ticket.sale();
            }
        }, "C").start();
    }
}

/**
 * Lock三部曲
 * 1.new ReentrantLock();
 * 2.Lock.lock() //加锁
 * 3.finally => lock.unlock() //解锁
 */
class Ticket2 {
    // 属性、方法
    private int number = 30;

    Lock lock = new ReentrantLock();

    // 卖票的方式
    public void sale() {

        lock.lock();//加锁


        try {
            //业务代码
            if (number > 0) {
                System.out.println(Thread.currentThread().getName() + "卖出了第" + (number--) + "票,剩余：" + number);
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            //解锁
            lock.unlock();
        }
    }
}
```

### synchronized 和 Lock 区别

1. synchronize 内置的Java关键字，Lock是一个Java类
2. synchronized 无法判断获取锁的状态，Lock 可以判断是否获取到了锁
3. synchronized 会自动释放锁，Lock必须要手动释放锁！如果不释放锁，**死锁**
4. synchronized 线程 1（获得锁，阻塞）、线程2（等待，傻傻的等）；Lock锁就不一定会等待下去；
5. synchronized 可重入锁，不可以中断的，非公平；Lock 可重入锁，可以判断锁，非公平（可以自己设置）
6. synchronized 适合锁少量的代码同步问题，Lock 适合锁大量的同步代码！

## 4、生产者和消费者问题

### 生产者和消费者问题 synchronized版 

```java
package com.ouwen.pc;

import com.sun.org.apache.bcel.internal.generic.NEW;

/**
 * 线程之间的通信问题：生产者与消费者问题！ 等待唤醒，通知唤醒
 * 线程交替执行 A B 操作同一个变量 num = 0
 * A num+1
 * B num-1
 * @author IRVING
 * @create 2021-04-04 15:10
 */
public class A {

    public static void main(String[] args) {
        Data data = new Data();

        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    data.increment();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        },"A").start();

        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    data.decrement();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        },"B").start();
    }
}

// 判断等待、业务、通知
//数字 资源类
class Data {
    private int number = 0;

    //+1
    public synchronized void increment() throws InterruptedException {
        if (number != 0) {
            //等待
            this.wait();
        }
        number++;
        System.out.println(Thread.currentThread().getName() + "=>" + number);
        // 通知其他线程，我+1完毕了
        this.notifyAll();
    }

    //-1
    public synchronized void decrement() throws InterruptedException {
        if (number == 0) {
            //等待
            this.wait();
        }
        number--;
        // 通知其他线程，我-1完毕了
        System.out.println(Thread.currentThread().getName() + "=>" + number);
        this.notifyAll();
    }
}
```

> 问题存在，A B C D 4个线程  虚假唤醒

![image-20210404152512622](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210404152512.png)

**if 改为 while 判断**

```java
package com.ouwen.pc;

import com.sun.org.apache.bcel.internal.generic.NEW;

/**
 * 线程之间的通信问题：生产者与消费者问题！ 等待唤醒，通知唤醒
 *
 * @author IRVING
 * @create 2021-04-04 15:10
 */
public class A {

    public static void main(String[] args) {
        Data data = new Data();

        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    data.increment();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        },"A").start();

        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    data.decrement();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        },"B").start();

        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    data.increment();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        },"C").start();

        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    data.decrement();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        },"D").start();
    }
}

// 判断等待、业务、通知
//数字 资源类
class Data {
    private int number = 0;

    //+1
    public synchronized void increment() throws InterruptedException {
        while (number != 0) {
            //等待
            this.wait();
        }
        number++;
        System.out.println(Thread.currentThread().getName() + "=>" + number);
        // 通知其他线程，我+1完毕了
        this.notifyAll();
    }

    //-1
    public synchronized void decrement() throws InterruptedException {
        while (number == 0) {
            //等待
            this.wait();
        }
        number--;
        // 通知其他线程，我-1完毕了
        System.out.println(Thread.currentThread().getName() + "=>" + number);
        this.notifyAll();
    }
}
```

### JUC版的生产者与消费者问题

![image-20210404153311646](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210404153425.png)

**通过Lock 找到 Condition**

![image-20210404160949444](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210404160949.png)

代码实现：

```java
package com.ouwen.pc;

import java.lang.reflect.Constructor;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

/**
 * @author IRVING
 * @create 2021-04-04 16:01
 */
public class B {

    public static void main(String[] args) {
        Data2 data = new Data2();

        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    data.increment();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "A").start();

        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    data.decrement();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "B").start();

        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    data.increment();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "C").start();

        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    data.decrement();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "D").start();
    }
}

// 判断等待、业务、通知
//数字 资源类
class Data2 {
    private int number = 0;

    Lock lock = new ReentrantLock();
    Condition condition = lock.newCondition();
    //condition.await();  //等待
    //condition.signalAll(); //唤醒全部


    //+1
    public void increment() throws InterruptedException {
        lock.lock();
        try {
            //业务代码
            while (number != 0) {
                //等待
                condition.await();
            }
            number++;
            System.out.println(Thread.currentThread().getName() + "=>" + number);
            // 通知其他线程，我+1完毕了
            condition.signalAll();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    //-1
    public void decrement() throws InterruptedException {
        lock.lock();
        try {
            while (number == 0) {
                //等待
                condition.await();
            }
            number--;
            // 通知其他线程，我-1完毕了
            System.out.println(Thread.currentThread().getName() + "=>" + number);
            condition.signalAll();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }

    }
}
```

**任何一个新的技术诞生，绝不是仅仅只是覆盖了原来的技术，一定存在优势和补充！**

### Condition 精准的通知和唤醒线程

<img src="https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210404161317.png" alt="image-20210404161317196"  />

代码测试：

```java
package com.ouwen.pc;

import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

/**
 * @author IRVING
 * @create 2021-04-04 16:14
 * A执行完调用B，B执行完调用C，C执行完调用A
 */
public class C {

    public static void main(String[] args) {

        Data3 data3 = new Data3();

        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                data3.printA();
            }
        }, "A").start();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                data3.printB();
            }
        }, "B").start();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                data3.printC();
            }
        }, "C").start();
    }
}

//资源类 Lock
class Data3 {

    private Lock lock = new ReentrantLock();
    Condition condition1 = lock.newCondition();
    Condition condition2 = lock.newCondition();
    Condition condition3 = lock.newCondition();
    private int number = 1; // 1A 2B 3C

    public void printA() {
        lock.lock();

        try {
            //业务，判断 -> 执行 -> 通知
            while (number != 1) {
                //等待
                condition1.await();
            }
            System.out.println(Thread.currentThread().getName() + "=>AAAAAAA");
            //唤醒，唤醒指定的人，B
            number = 2;
            condition2.signal();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    public void printB() {
        lock.lock();

        try {
            //业务，判断 -> 执行 -> 通知
            while (number != 2) {
                //等待
                condition2.await();
            }
            System.out.println(Thread.currentThread().getName() + "=>BBBBBB");
            //唤醒，唤醒指定的人，C
            number = 3;
            condition3.signal();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    public void printC() {
        lock.lock();

        try {
            //业务，判断 -> 执行 -> 通知
            while (number != 3) {
                //等待
                condition3.await();
            }
            System.out.println(Thread.currentThread().getName() + "=>CCCCCCCC");
            //唤醒，唤醒指定的人，A
            number = 1;
            condition1.signal();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
}
```

## 5、8锁现象

任何判断锁的是谁！永远的知道什么是锁，锁到底锁的是谁！

**深刻理解我们的锁	**

```java
package com.ouwen.lock8;

import java.util.concurrent.TimeUnit;

/**
 * 8锁，就是关于锁的8个问题
 * 1、标准情况下，两个线程先打印 发短信还是 打电话？ 1发短信 2打电话
 * 1、sendSms延迟4秒，两个线程先打印 发短信还是 打电话？ 1发短信 2打电话
 * @author IRVING
 * @create 2021-04-04 16:30
 */
public class Test1 {

    public static void main(String[] args) {
        Phone phone = new Phone();

        //锁的存在
        new Thread(()->{
            phone.sendSms();
        }).start();

        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        new Thread(()->{
            phone.call();
        }).start();
    }
}

class Phone {

    // synchronized 锁的对象是方法的调用者！
    // 两个方法用的是同一所，谁先拿到谁执行！
    public synchronized void sendSms() {
        try {
            TimeUnit.SECONDS.sleep(4);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("发短信");
    }

    public synchronized void call() {
        System.out.println("打电话");
    }
}
```

```java
package com.ouwen.lock8;

import java.util.concurrent.TimeUnit;

/**
 * 3. 增加了一个普通方法！先执行发短信还是hello？  普通方法hello
 * 4. 两个对象，两个同步方法，发短信还是打电话？  //打电话
 * @author IRVING
 * @create 2021-04-04 16:36
 */
public class Test2  {

    public static void main(String[] args) {
        //两个不同的对象 两把锁
        Phone2 phone1 = new Phone2();
        Phone2 phone2 = new Phone2();

        //锁的存在
        new Thread(()->{
            phone1.sendSms();
        },"A").start();

        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        new Thread(()->{
            phone2.call();
        },"B").start();
    }
}

class Phone2 {

    // synchronized 锁的对象是方法的调用者！
    // 两个方法用的是同一锁，谁先拿到谁执行！
    public synchronized void sendSms() {
        try {
            TimeUnit.SECONDS.sleep(4);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("发短信");
    }

    public synchronized void call() {
        System.out.println("打电话");
    }

    // 这里没有锁！不是同步方法！不受锁的影响
    public void hello(){
        System.out.println("hello");
    }
}
```

```java
package com.ouwen.lock8;

import java.util.concurrent.TimeUnit;

/**
 * 5.增加两个静态同步方法，只有一个对象，先打印 发短信？打电话？ 发短信
 * 6.增加两个静态同步方法，两个对象！，先打印 发短信？打电话？ 发短信
 * @author IRVING
 * @create 2021-04-04 17:22
 */
public class Test3 {

    public static void main(String[] args) {
        //两个不同的对象 类模板Class只有一个 static，锁的是Class
        Phone3 phone1 = new Phone3();
        Phone3 phone2 = new Phone3();

        //锁的存在
        new Thread(() -> {
            phone1.sendSms();
        }, "A").start();

        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        new Thread(() -> {
            phone2.call();
        }, "B").start();
    }
}

class Phone3 {

    // synchronized 锁的对象是方法的调用者！
    // staic静态方法
    // 类一加载就有了！锁的是Class Phone3.class
    public static synchronized void sendSms() {
        try {
            TimeUnit.SECONDS.sleep(4);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("发短信");
    }

    public static synchronized void call() {
        System.out.println("打电话");
    }
}
```

```java
package com.ouwen.lock8;

import java.util.concurrent.TimeUnit;

/**
 * 7. 一个静态同步方法，一个普通同步方法 一个对象 先打印 发短信？打电话？ 打电话
 * 8. 一个静态同步方法，一个普通同步方法 两个对象 先打印 发短信？打电话？ 打电话
 *
 * @author IRVING
 * @create 2021-04-04 17:27
 */
public class Test4 {

    public static void main(String[] args) {
        //两个不同的对象 类模板Class只有一个 static，锁的是Class
        Phone4 phone1 = new Phone4();
        Phone4 phone2 = new Phone4();

        //锁的存在
        new Thread(() -> {
            phone1.sendSms();
        }, "A").start();

        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        new Thread(() -> {
            phone2.call();
        }, "B").start();
    }
}

class Phone4 {

    // staic静态同步方法  锁的是Class
    public static synchronized void sendSms() {
        try {
            TimeUnit.SECONDS.sleep(4);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("发短信");
    }

    //普通同步方法 锁的是调用对象
    public synchronized void call() {
        System.out.println("打电话");
    }
}
```

> 小结

new this 具体的一个对象

static Class 唯一的模板

## 6、集合类不安全

### List不安全

```java
package com.ouwen.unsafe;

import java.util.*;
import java.util.concurrent.CopyOnWriteArrayList;

/**
 * java.util.ConcurrentModificationException 并发修改异常;
 *
 * @author IRVING
 * @create 2021-04-04 17:34
 */
public class ListTest {

    public static void main(String[] args) {
        // 并发下 ArrayList 不安全的
        /**
         * 解决方案：
         * 1、List<String> list = new Vector<>();
         * 2、List<String> list = Collections.synchronizedList(new ArrayList<>());
         * 3、List<String> list = new CopyOnWriteArrayList<>();
         */
        // CopyOnWriter COW 计算机程序设计领域的一种优化策略：
        // 多个线程调用的时候，list，读取时是固定的，写入（覆盖）
        // 在写入的时候避免覆盖，造成数据问题  -- 读写分离
        // CopyOnWriterList 比 Vector NB在哪里？ => 前者效率高
        List<String> list = new CopyOnWriteArrayList<>();
        for (int i = 0; i < 10; i++) {
            new Thread(() -> {
                list.add(UUID.randomUUID().toString().substring(0, 5));
                System.out.println(list);
            }, String.valueOf(i)).start();
        }
    }
}
```

学习方法推荐：1、先会用、2、货比3家，寻找其他解决方案，3、分析源码！

### Set不安全

```java
package com.ouwen.unsafe;

import java.util.Collections;
import java.util.HashSet;
import java.util.Set;
import java.util.UUID;
import java.util.concurrent.CopyOnWriteArraySet;

/**
 * 同理可证：java.util.ConcurrentModificationException 并发修改异常
 * @author IRVING
 * @create 2021-04-04 17:49
 */
public class SetList {
    public static void main(String[] args) {
        //Set<String> set = new HashSet<>();
        //Set<String> set = Collections.synchronizedSet(new HashSet<>());
        Set<String> set = new CopyOnWriteArraySet<>();
        for (int i = 0; i < 300; i++) {
            new Thread(() -> {
                set.add(UUID.randomUUID().toString().substring(0, 5));
                System.out.println(set);
            }, String.valueOf(i)).start();
        }
    }
}
```

**HashSet 底层是什么？**

```java
public HashSet() {
    map = new HashMap<>();
}

//add set 本质就是map key是无法重复的！！
public boolean add(E e) {
    return map.put(e, PRESENT)==null;
}

private static final Object PRESENT = new Object(); //常量 不变的值！
```

### Map 不安全

```java
package com.ouwen.unsafe;

import java.util.Collections;
import java.util.HashMap;
import java.util.Map;
import java.util.UUID;
import java.util.concurrent.ConcurrentHashMap;

/**
 * @author IRVING
 * @create 2021-04-04 17:57
 */
public class MapTest {

    public static void main(String[] args) {
        // map 是这样的用的吗？ 不是，工作中不用HashMap()
        //  默认等价于什么 new HashMap<>(16,0.75f);
        //Map<String,String> map =  new HashMap<>();
        //Map<String,String> map = Collections.synchronizedMap(new HashMap<>());
        // 研究ConcurrentHashMap原理
        Map<String,String> map = new ConcurrentHashMap<>();
        for (int i = 0; i < 30; i++) {
            new Thread(()->{
                map.put(Thread.currentThread().getName(), UUID.randomUUID().toString().substring(0,5));
                System.out.println(map);
            },String.valueOf(i)).start();
        }
    }
}
```

## 7、Callable（简单）

![image-20210404181130827](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210404181130.png)

1. 可以有返回值
2. 可以抛出异常
3. 方法不同 ，run()/call()

**代码测试：**

```java
package com.ouwen.callable;

import java.util.concurrent.Callable;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.FutureTask;

/**
 * 启动Callable
 *
 * @author IRVING
 * @create 2021-04-04 18:13
 */
public class CallableTest {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        //怎么启动callable
        // 1.new Thread(new Runnable()).start();
        // 2.new Thread(new FutureTask<V>()).start();
        // 3.new Thread(new FutureTask<V>(new Callable())).start();


        MyThread myThread = new MyThread();
        //适配类 FutureTask
        FutureTask<String> task = new FutureTask<>(myThread);
        new Thread(task, "A").start();
        new Thread(task, "B").start(); //结果会被缓存，效率高

        String s = task.get(); // 这个get方法可能会产生阻塞！ 把它放到最后，或者使用异步通信来处理
        System.out.println(s);
    }
}

class MyThread implements Callable<String> {
    @Override
    public String call() throws Exception {
        System.out.println(Thread.currentThread().getName()+"call()");
        return "123";
    }
}

```

细节：

1. 有缓存
2. 结果可能需要等待，会阻塞！

## 8、常用的辅助类

### 8.1、CountDownLatch

![image-20210404211341279](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210404211341.png)

```java
package com.ouwen.add;

import java.util.concurrent.CountDownLatch;

/**
 * 计数器
 * @author IRVING
 * @create 2021-04-04 21:14
 */
public class CountDownLatchDemo {

    public static void main(String[] args) throws InterruptedException {
        //总数是6
        CountDownLatch countDownLatch = new CountDownLatch(6);

        for (int i = 0; i < 6; i++) {
            new Thread(()->{
                System.out.println(Thread.currentThread().getName()+"Go Out");
                countDownLatch.countDown(); //数量-1
            },String.valueOf(i)).start();
        }

        countDownLatch.await(); //等待计数器归零，再向下执行

        System.out.println("关门");
    }

}
```

原理：

==countDownLatch.countDown()==  //数量-1

==countDownLatch.await()==  //等待计数器归零，然后再向下执行

每次有线程调用countDownLatch()数量-1，假设计数器变为0，countDownLatch.await()就会被唤醒，继续往下执行

### 8.2、CyclicBarrier

![image-20210404212739912](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210404212740.png)

加法计数器

```java
package com.ouwen.add;

import java.util.concurrent.BrokenBarrierException;
import java.util.concurrent.CyclicBarrier;

/**
 * @author IRVING
 * @create 2021-04-04 21:28
 */
public class CycliBarrierDemo {

    public static void main(String[] args) {
        /**
         * 集齐七颗龙珠，召唤神龙
         */
        //召唤龙珠的线程
        CyclicBarrier barrier = new CyclicBarrier(7,()->{
            System.out.println("召唤神龙成功");
        });
        
        for (int i = 0; i < 7; i++) {
            int temp = i;
            // lambda能操作i吗
            new Thread(()->{
                System.out.println(Thread.currentThread().getName()+"收集"+temp+"个龙珠");
                try {
                    barrier.await(); //等待
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } catch (BrokenBarrierException e) {
                    e.printStackTrace();
                }
            }).start();
        }
    }
}
```

### 8.3、Semaphore

![image-20210404213555658](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210404213555.png)

```java
package com.ouwen.add;

import java.util.concurrent.Semaphore;
import java.util.concurrent.TimeUnit;

/**
 * @author IRVING
 * @create 2021-04-04 21:36
 */
public class SemaphoreDemo {

    public static void main(String[] args) {
        //线程数量：停车位
        Semaphore semaphore = new Semaphore(3);

        for (int i = 0; i < 6; i++) {
            new Thread(() -> {
                //acquire()得到
                try {
                    semaphore.acquire();
                    System.out.println(Thread.currentThread().getName() + "抢到车位");
                    TimeUnit.SECONDS.sleep(2);
                    System.out.println(Thread.currentThread().getName() + "离开车位");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    //release()释放
                    semaphore.release();
                }
            },String.valueOf(i)).start();
        }
    }
}
```

原理：

==semaphore.acquire();==  // 获得，假设已经满了，等待，等待被释放为止！

==semaphore.release();==  // 释放，会将当前的信号量释放+1，然后唤醒等待的线程！

作用：多个共享资源互斥的使用！并发限流，控制并发的线程数

## 9、读写锁

ReadWriteLock

![image-20210405142730413](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210405142737.png)

```java
package com.ouwen.rw;

import java.util.HashMap;
import java.util.Map;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReadWriteLock;
import java.util.concurrent.locks.ReentrantReadWriteLock;

/**
 * 独占锁（写锁） 一次只能被一个线程占有
 * 共享锁（读锁）  多个线程可以同时占有 
 * ReadWriteLock
 * 读-读 可以共存！
 * 读-写 不能共存！
 * 写-写 不能共存！
 * @author IRVING
 * @create 2021-04-05 14:28
 */
public class ReadWriteLockDemo {
    public static void main(String[] args) {
        MyCacheLock myCache = new MyCacheLock();

        //写入
        for (int i = 0; i < 5; i++) {
            final int temp = i;
            new Thread(() -> {
                myCache.put(temp + "", temp);
            }, String.valueOf(i)).start();
        }

        //读取
        for (int i = 0; i < 5; i++) {
            final int temp = i;
            new Thread(() -> {
                myCache.get(temp + "");
            }, String.valueOf(i)).start();
        }
    }
}

/**
 * 自定义缓存
 */
class MyCache {
    private volatile Map<String, Object> map = new HashMap<>();

    // 存，写
    public void put(String key, Object value) {
        System.out.println(Thread.currentThread().getName() + "写入" + key);
        map.put(key, value);
        System.out.println(Thread.currentThread().getName() + "写入完毕");
    }

    // 取，读
    public void get(String key) {
        System.out.println(Thread.currentThread().getName() + "读取" + key);
        Object o = map.get(key);
        System.out.println(Thread.currentThread().getName() + "读取完毕");
    }
}

class MyCacheLock {
    private volatile Map<String, Object> map = new HashMap<>();
    // 读写锁，更加细粒度的控制
    private ReadWriteLock readWriteLock = new ReentrantReadWriteLock();

    // 存，写入的时候，只希望同时只有一个线程写
    public void put(String key, Object value) {
        readWriteLock.writeLock().lock();

        try {
            System.out.println(Thread.currentThread().getName() + "写入" + key);
            map.put(key, value);
            System.out.println(Thread.currentThread().getName() + "写入完毕");
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            readWriteLock.writeLock().unlock();
        }

    }

    // 取，读
    public void get(String key) {
        readWriteLock.readLock().lock();
        try {
            System.out.println(Thread.currentThread().getName() + "读取" + key);
            Object o = map.get(key);
            System.out.println(Thread.currentThread().getName() + "读取完毕");
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            readWriteLock.readLock().unlock();
        }

    }
}
```

## 10、阻塞队列

![image-20210405160056615](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210405160056.png)

### 阻塞队列

![image-20210405160225524](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210405160225.png)

![image-20210405160713694](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210405160713.png)

### **BlockingQueue** 

不是新的东西，属于Collection集合框架下

![image-20210405160927280](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210405160927.png)

什么情况下我们会使用阻塞队列：多线程并发处理，线程池！

### **学会使用队列**

添加、移除

**四组API**

| 方式         | 有返回值，抛出异常 | 有返回值，不抛出异常 | 阻塞等待 | 超时等待  |
| ------------ | ------------------ | -------------------- | -------- | --------- |
| 添加         | add                | offer()              | put()    | offer(,,) |
| 移除         | remove             | foll()               | take()   | foll(,)   |
| 检测队首元素 | element            | peek                 | -        | -         |

```java
/**
 * 抛出异常
 */
public static void test1() {
    //队列的大小
    ArrayBlockingQueue blockingQueue = new ArrayBlockingQueue(3);
    System.out.println(blockingQueue.add("a"));
    System.out.println(blockingQueue.add("b"));
    System.out.println(blockingQueue.add("c"));
    //java.lang.IllegalStateException: Queue full 抛出异常！队列已满
    //System.out.println(blockingQueue.add("d"));

    System.out.println("==========");

    System.out.println(blockingQueue.remove());
    System.out.println(blockingQueue.remove());
    System.out.println(blockingQueue.remove());
    //java.util.NoSuchElementException 抛出异常！没有元素
    System.out.println(blockingQueue.remove());
}
```

```java
/**
 * 有返回值，不抛出异常
 */
public static void test2(){
    ArrayBlockingQueue blockingQueue = new ArrayBlockingQueue(3);

    System.out.println(blockingQueue.offer("a"));
    System.out.println(blockingQueue.offer("b"));
    System.out.println(blockingQueue.offer("c"));
    //System.out.println(blockingQueue.offer("d"));  //返回false

    System.out.println("========");

    System.out.println(blockingQueue.poll());
    System.out.println(blockingQueue.poll());
    System.out.println(blockingQueue.poll());
    System.out.println(blockingQueue.poll());  //返回null
}
```

```java
/**
 * 阻塞等待
 */
public static void test3() throws InterruptedException {
    ArrayBlockingQueue<Object> blockingQueue = new ArrayBlockingQueue<>(3);

    blockingQueue.put("a");
    blockingQueue.put("b");
    blockingQueue.put("c");
    //blockingQueue.put("d"); //程序一直阻塞
    
    System.out.println("========");

    System.out.println(blockingQueue.take());
    System.out.println(blockingQueue.take());
    System.out.println(blockingQueue.take());
    System.out.println(blockingQueue.take()); //程序一直阻塞
}
```

```java
/**
 * 超时等待
 */
public static void test4() throws InterruptedException {
    ArrayBlockingQueue<Object> blockingQueue = new ArrayBlockingQueue<>(3);

    System.out.println(blockingQueue.offer("a"));
    System.out.println(blockingQueue.offer("b"));
    System.out.println(blockingQueue.offer("c"));
    //System.out.println(blockingQueue.offer("d",2, TimeUnit.SECONDS));  //超时等待2秒

    System.out.println("=========");

    System.out.println(blockingQueue.poll());
    System.out.println(blockingQueue.poll());
    System.out.println(blockingQueue.poll());
    System.out.println(blockingQueue.poll(2,TimeUnit.SECONDS)); //超时等待2秒
}
```

### SynchronousQueue

**同步队列**

进去一个元素，必须等待取出来之后，才能再往里面放一个元素！

put、take

```java
package com.ouwen.bq;

import java.util.concurrent.BlockingQueue;
import java.util.concurrent.SynchronousQueue;
import java.util.concurrent.TimeUnit;

/**
 * 同步队列
 * 和其他的BlockingQueue不一样，SynchronousQueue 不存储元素
 * put了一个元素，必须从里面先take出来，否则无法继续往里面put值
 * @author IRVING
 * @create 2021-04-05 17:03
 */
public class SynchronousQueueTest {

    public static void main(String[] args) {
        BlockingQueue<String> blockingQueue = new SynchronousQueue<>();//同步队列

        new Thread(()->{
            try {
                System.out.println(Thread.currentThread().getName()+"put 1");
                blockingQueue.put("1");
                System.out.println(Thread.currentThread().getName()+"put 2");
                blockingQueue.put("2");
                System.out.println(Thread.currentThread().getName()+"put 3");
                blockingQueue.put("3");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        },"t1").start();


        new Thread(()->{
            try {
                TimeUnit.SECONDS.sleep(2);
                System.out.println(Thread.currentThread().getName()+blockingQueue.take());
                TimeUnit.SECONDS.sleep(2);
                System.out.println(Thread.currentThread().getName()+blockingQueue.take());
                TimeUnit.SECONDS.sleep(2);
                System.out.println(Thread.currentThread().getName()+blockingQueue.take());
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        },"t2").start();
    }
}
```

## 11、线程池（重点）

线程池：三大方法、七大参数、四种拒绝策略

### 池化技术

程序的运行，本质：占用系统的资源！优化资源的使用！=>池化技术

线程池、连接池、内存池、对象池....创建、销毁。十分浪费资源

池化技术：事先准备好一些资源，有人要用，就到我这里来拿，用完之后还给我。

### 线程池的好处

1. 降低资源的消耗
2. 提高响应的速度
3. 方便管理

==**线程复用，可以控制最大并发数、管理线程**==

### 线程池：三大方法

![image-20210405171659980](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210405171700.png)

```java
package com.ouwen.pool;

import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

/**
 * Executors 工具类 3大方法
 * @author IRVING
 * @create 2021-04-05 17:19
 */
public class Demo01 {
    public static void main(String[] args) {
        //ExecutorService threadPool =  Executors.newSingleThreadExecutor(); //单个线程
        //ExecutorService threadPool =  Executors.newFixedThreadPool(5); //创建一个固定大小的线程池
        ExecutorService threadPool =  Executors.newCachedThreadPool(); //可伸缩的，缓存线程池


        try {
            for (int i = 0; i < 10; i++) {
                threadPool.execute(()->{
                    System.out.println(Thread.currentThread().getName());
                });
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            threadPool.shutdown();  //关闭线程池
        }
    }
}
```

### 七大参数

**源码分析：**

```java
public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService
        (new ThreadPoolExecutor(1, 1,
                                0L, TimeUnit.MILLISECONDS,
                                new LinkedBlockingQueue<Runnable>()));
}

public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads,
                                  0L, TimeUnit.MILLISECONDS,
                                  new LinkedBlockingQueue<Runnable>());
}

public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                  60L, TimeUnit.SECONDS,
                                  new SynchronousQueue<Runnable>());
}

//本质都是ThreadPoolExecutor()
public ThreadPoolExecutor(int corePoolSize,  //核心线程池大小
                          int maximumPoolSize, //最大核心线程池大小
                          long keepAliveTime,  //超时了没有人调用就会释放
                          TimeUnit unit,  //超时单位
                          BlockingQueue<Runnable> workQueue,  //阻塞队列
                          ThreadFactory threadFactory,  //线程工厂，创建线程的，一般不用动
                          RejectedExecutionHandler handler  //拒绝策略) {
    if (corePoolSize < 0 ||
        maximumPoolSize <= 0 ||
        maximumPoolSize < corePoolSize ||
        keepAliveTime < 0)
        throw new IllegalArgumentException();
    if (workQueue == null || threadFactory == null || handler == null)
        throw new NullPointerException();
    this.corePoolSize = corePoolSize;
    this.maximumPoolSize = maximumPoolSize;
    this.workQueue = workQueue;
    this.keepAliveTime = unit.toNanos(keepAliveTime);
    this.threadFactory = threadFactory;
    this.handler = handler;
}
```

![image-20210405181241166](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210405181241.png)

![image-20210405181515918](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210405190839.png)

### 手动创建线程池

```java
package com.ouwen.pool;

import java.util.concurrent.*;

/**
 * Executors 工具类 3大方法
 * @author IRVING
 * @create 2021-04-05 17:19
 */
public class Demo01 {
    public static void main(String[] args) {
        //自定义线程池！工作中 ThreadPoolExecutor
        ExecutorService threadPool = new ThreadPoolExecutor(2,
                5,
                3,
                TimeUnit.SECONDS,
                new LinkedBlockingDeque<>(3),
                Executors.defaultThreadFactory(),
                /**
                 * 四种拒绝策略
                 * 1.new ThreadPoolExecutor.AbortPolicy());  // 银行满了，还有人进来，不处理这个人的，抛出异常
                 * 2.new ThreadPoolExecutor.CallerRunsPolicy());  // 哪来的去哪里
                 * 3.new ThreadPoolExecutor.DiscardPolicy());  // 队列满了，不会抛出异常，丢弃
                 * 4.new ThreadPoolExecutor.DiscardOldestPolicy());  // 队列满了，丢弃最早的任务，添加新任务
                 */
                new ThreadPoolExecutor.DiscardOldestPolicy());  // 队列满了，丢弃最早的任务，添加新任务

        try {
            // 最大承载：Deque + max
            // 抛出 java.util.concurrent.RejectedExecutionException
            for (int i = 0; i < 9; i++) {
                threadPool.execute(()->{
                    System.out.println(Thread.currentThread().getName());
                });
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            threadPool.shutdown();  //关闭线程池
        }
    }
}
```

### 四种拒绝策略

![image-20210405185214360](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210405185214.png)

```java
/**
 * 四种拒绝策略
 * 1.new ThreadPoolExecutor.AbortPolicy());  // 银行满了，还有人进来，不处理这个人的，抛出异常
 * 2.new ThreadPoolExecutor.CallerRunsPolicy());  // 哪来的去哪里
 * 3.new ThreadPoolExecutor.DiscardPolicy());  // 队列满了，不会抛出异常，丢弃
 * 4.new ThreadPoolExecutor.DiscardOldestPolicy());  // 队列满了，丢弃最早的任务，添加新任务
 */
```

### 小结和扩展

最大线程池大小如何去设置！

了解：IO密集型，CPU密集型：（调优）！

```java
package com.ouwen.pool;

import java.util.concurrent.*;

/**
 * @author IRVING
 * @create 2021-04-05 17:19
 */
public class Demo01 {
    public static void main(String[] args) {
        //自定义线程池！工作中 ThreadPoolExecutor

        /**
         * 最大线程到底该如何定义？
         * 1. CPU密集型，几核，就是几，可以保持CPU效率最高
         * 2. IO密集型，判断你程序中十分耗IO的线程
         * 程序  15个大型任务  io十分占用资源！ -> 30个
         */
        
        //获取CPU的核心数
        System.out.println(Runtime.getRuntime().availableProcessors());
        
        ExecutorService threadPool = new ThreadPoolExecutor(2,
                Runtime.getRuntime().availableProcessors(),
                3,
                TimeUnit.SECONDS,
                new LinkedBlockingDeque<>(3),
                Executors.defaultThreadFactory(),
                new ThreadPoolExecutor.DiscardOldestPolicy()); 
        try {
            // 最大承载：Deque + max
            // 抛出 java.util.concurrent.RejectedExecutionException
            for (int i = 0; i < 9; i++) {
                threadPool.execute(()->{
                    System.out.println(Thread.currentThread().getName());
                });
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            threadPool.shutdown();  //关闭线程池
        }
    }
}
```

## 12、四大函数式接口（重点、必需掌握）

新时代的程序员：lambda表达式、链式编程、函数式接口、Stream流式计算

### 函数式接口：只有一个方法的接口

```java
@FunctionalInterface
public interface Runnable {
    public abstract void run();
}

//超级多的FunctionInterface
//简化编程模型，在新版本的框架底层大量应用！
//forEach(消费者类型的函数式接口)
```

![image-20210405193839177](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210405193839.png)	

**代码测试：**

### Function接口

函数型接口

<img src="https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210405195016.png" alt="image-20210405195016832" style="zoom:50%;" />	

```java
package com.ouwen.function;

import java.util.function.Function;

/**
 * Function 函数型接口，有一个输入参数，有一个输出参数
 * 只要是函数型接口，都可以用lambda表达式简化
 *
 * @author IRVING
 * @create 2021-04-05 19:45
 */
public class Demo01 {

    public static void main(String[] args) {
        //Function<String, String> function = new Function<String, String>() {
        //    @Override
        //    public String apply(String s) {
        //        return s;
        //    }
        //};

       	//使用lambda表达式简化
        Function<String, String> function = (str) -> {return str;};

        System.out.println(function.apply("asd"));
    }
}
```

### Predicate接口

断定型接口

<img src="https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210405195655.png" alt="image-20210405195655121" style="zoom:50%;" />	

```java
package com.ouwen.function;

import java.util.function.Predicate;

/**
 * 断定型接口：有一个输入参数，返回值只能是布尔值！
 * @author IRVING
 * @create 2021-04-05 19:51
 */
public class Demo02 {
    public static void main(String[] args) {
        //判断字符是否为空
        //Predicate<String> predicate = new Predicate<String>() {
        //    @Override
        //    public boolean test(String s) {
        //        return s.isEmpty();
        //    }
        //};

        Predicate<String> predicate = str -> str.isEmpty();

        System.out.println(predicate.test(""));
    }
}
```

### Consumer接口

消费型接口

<img src="https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210405200414.png" alt="image-20210405200414021" style="zoom:50%;" />	

```java
package com.ouwen.function;

import com.sun.org.apache.bcel.internal.generic.NEW;

import java.util.function.Consumer;

/**
 * 消费性接口：一个输入参数，没有返回值
 * @author IRVING
 * @create 2021-04-05 20:00
 */
public class Demo03 {

    public static void main(String[] args) {
        //Consumer<String> consumer = new Consumer<String>() {
        //    @Override
        //    public void accept(String s) {
        //        System.out.println(s);
        //    }
        //};
        
        Consumer<String> consumer = str -> System.out.println(str);

        consumer.accept("asd");
    }
}
```

### Supplier接口

供给型接口

<img src="https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210405200437.png" alt="image-20210405200437068" style="zoom:50%;" />	

```java
package com.ouwen.function;

import java.util.function.Supplier;

/**
 * Supplier供给型接口 没有参数，只有返回值
 * @author IRVING
 * @create 2021-04-05 20:03
 */
public class Demo04 {
    public static void main(String[] args) {
        //Supplier<Integer> supplier = new Supplier<Integer>() {
        //    @Override
        //    public Integer get() {
        //        return 1024;
        //    }
        //};

        Supplier<Integer> supplier = () -> 1024;

        System.out.println(supplier.get());
    }
}
```

## 13、Stream流式计算

### 什么是流式计算？

大数据时代：存储+计算

集合是用来存储东西的

计算都应该交给流来操作！

<img src="https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210405225822.png" alt="image-20210405225822466" style="zoom:67%;" />

```java
package com.ouwen.stream;

import java.util.Arrays;
import java.util.Comparator;
import java.util.List;
import java.util.function.Function;

/**
 * 现有5个用户，筛选：
 * 1. ID偶数的
 * 2. 年纪大于23岁
 * 3. 用户名转为大写字母
 * 4. 用户名字母倒排序
 * 5. 只输出一个用户
 * <p>
 * * @author IRVING
 * * @create 2021-04-05 22:49
 */
public class Test {

    public static void main(String[] args) {
        User u1 = new User(1, "a", 21);
        User u2 = new User(2, "b", 22);
        User u3 = new User(3, "c", 23);
        User u4 = new User(4, "d", 24);
        User u5 = new User(6, "e", 26);

        List<User> users = Arrays.asList(u1, u2, u3, u4, u5);

        users.stream()
                .filter(u -> u.getId() % 2 == 0)
                .filter(u -> u.getAge() > 23)
                .peek(user -> user.setName(user.getName().toUpperCase()))
                .sorted(Comparator.comparing(User::getName, Comparator.reverseOrder()))
                .limit(1)
                .forEach(System.out::println);
    }
}

```

## 14、ForkJoin

分支合并

### 什么是ForkJoin？

ForkJoin 在JDK1.7，并行执行任务！提高效率，大数据量！

<img src="https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210406124700.png" alt="image-20210406124653585" style="zoom: 50%;" />	

> ForkJoin 特点：工作窃取

这个里面维护的都是双端队列

<img src="https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210406125200.png" alt="image-20210406125200490" style="zoom:50%;" />	

### 如何使用ForkJoin?

<img src="https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210406130207.png" alt="image-20210406130207157" style="zoom:50%;" />	

<img src="https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210406132051.png" alt="image-20210406132051647" style="zoom:50%;" />	

## 15、异步回调

Future 设计的初衷：对将来的某个事件的结果进行建模

<img src="https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210406221746.png" alt="image-20210406221739820" style="zoom:67%;" />	

```java
package com.ouwen.future;

import java.util.concurrent.CompletableFuture;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.TimeUnit;

/**
 * 异步调用：CompletableFuture
 * 异步执行
 * 成功回调
 * 失败回调
 *
 * @author IRVING
 * @create 2021-04-06 22:18
 */
public class Demo01 {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        //发起一个请求
        //CompletableFuture<Void> completableFuture = CompletableFuture.runAsync(() -> {
        //    try {
        //        TimeUnit.SECONDS.sleep(5);
        //    } catch (InterruptedException e) {
        //        e.printStackTrace();
        //    }
        //    System.out.println(Thread.currentThread().getName() + "runAsync=>Void");
        //});
        //
        //System.out.println("1111");
        //
        //completableFuture.get(); //获取阻塞执行结果

        //有返回值的
        CompletableFuture<Integer> completableFuture = CompletableFuture.supplyAsync(() -> {
            System.out.println(Thread.currentThread().getName() + "runAsync=>Integer");
            int i = 10/0;
            return 1024;
        });

        completableFuture.whenComplete((t,u)->{
            System.out.println(t); //正常的返回结果
            System.out.println(u); //错误信息 java.util.concurrent.CompletionException: java.lang.ArithmeticException: / by zero
        }).exceptionally((e)->{
            System.out.println(e.getMessage()); //可以获取到错误的返回结果
            return 233;
        });
    }
}
```

## 16、JMM

### 请你谈谈你对Volatile的理解

Volatile 是Java 虚拟机提供的**轻量级的同步机制**

1. 保证可见性
2. 不保证原子性
3. 禁止指令重排

### 什么是JMM

JMM：java内存模型，不存在的东西，概念！约定！

**关于JMM一些同步的约定：**

1. 线程解锁前，必须把共享变量立刻刷回主内存
2. 线程加锁前，必须读取主内存中的最新值到工作内存中！
3. 加锁和解锁必须是同一把锁！

线程  **工作内存、主内存**

### 8种操作

![image-20210406224257522](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210406224257.png)

![image-20210406224329039](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210406224329.png)

**内存交互操作有8种，虚拟机实现必须保证每一个操作都是原子的，不可在分的（对于double和long类型的变量来说，load、store、read和write操作在某些平台上允许例外）**

- lock   （锁定）：作用于主内存的变量，把一个变量标识为线程独占状态
- unlock （解锁）：作用于主内存的变量，它把一个处于锁定状态的变量释放出来，释放后的变量才可以被其他线程锁定
- read  （读取）：作用于主内存变量，它把一个变量的值从主内存传输到线程的工作内存中，以便随后的load动作使用
- load   （载入）：作用于工作内存的变量，它把read操作从主存中变量放入工作内存中
- use   （使用）：作用于工作内存中的变量，它把工作内存中的变量传输给执行引擎，每当虚拟机遇到一个需要使用到变量的值，就会使用到这个指令
- assign （赋值）：作用于工作内存中的变量，它把一个从执行引擎中接受到的值放入工作内存的变量副本中
- store  （存储）：作用于主内存中的变量，它把一个从工作内存中一个变量的值传送到主内存中，以便后续的write使用
- write 　（写入）：作用于主内存中的变量，它把store操作从工作内存中得到的变量的值放入主内存的变量中

**JMM对这八种指令的使用，制定了如下规则：**

- 不允许read和load、store和write操作之一单独出现。即使用了read必须load，使用了store必须write

- 不允许线程丢弃他最近的assign操作，即工作变量的数据改变了之后，必须告知主存
- 不允许一个线程将没有assign的数据从工作内存同步回主内存
- 一个新的变量必须在主内存中诞生，不允许工作内存直接使用一个未被初始化的变量。就是怼变量实施use、store操作之前，必须经过assign和load操作
- 一个变量同一时间只有一个线程能对其进行lock。多次lock后，必须执行相同次数的unlock才能解锁
- 如果对一个变量进行lock操作，会清空所有工作内存中此变量的值，在执行引擎使用这个变量前，必须重新load或assign操作初始化变量的值
- 如果一个变量没有被lock，就不能对其进行unlock操作。也不能unlock一个被其他线程锁住的变量
- 对一个变量进行unlock操作之前，必须把此变量同步回主内存



问题：程序不知道主内存的值被修改过了！！

![image-20210406224933254](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210406224933.png)

```java
package com.ouwen.volatile1;

import java.util.concurrent.TimeUnit;

/**
 * @author IRVING
 * @create 2021-04-06 22:50
 */
public class JMMDemo {

    private static int num = 0;
    public static void main(String[] args) throws InterruptedException {
        new Thread(()->{
            while (num ==  0){ //线程1对主内存的变化不知道
                
            }
        }).start();

        TimeUnit.SECONDS.sleep(1);

        num = 1;
        System.out.println(num);
    }
}
```

## 17、Volatile

### 保证可见性

```java
package com.ouwen.volatile1;

import java.util.concurrent.TimeUnit;

/**
 * @author IRVING
 * @create 2021-04-06 22:50
 */
public class JMMDemo {
    /*
      不加 volatile 程序就会死循环
      加 volatile 可以保证可见性
     */
    private volatile static int num = 0;
    public static void main(String[] args) throws InterruptedException {
        new Thread(()->{
            while (num ==  0){ //线程1对主内存的变化不知道

            }
        }).start();

        TimeUnit.SECONDS.sleep(1);

        num = 1;
        System.out.println(num);
    }
}
```

### 不保证原子性

原子性：不可分割

线程A在执行任务的时候，不能被打扰的，也不能被分割，要么同时成功，要么同时失败！

```java
package com.ouwen.volatile1;

/**
 * @author IRVING
 * @create 2021-04-06 22:55
 */
public class Demo2 {

    //volatile不保证原子性
    private volatile static int num = 0;

    public static void add() {
        num++; //不是一个原子性操作
    }

    public static void main(String[] args) {
        //理论上num结果为2万
        for (int i = 0; i < 20; i++) {
            new Thread(() -> {
                for (int i1 = 0; i1 < 1000; i1++) {
                    add();
                }
            }).start();
        }

        while (Thread.activeCount() > 2) { //main gc
            Thread.yield();
        }

        System.out.println(Thread.currentThread().getName() + "==>" + num);
    }
}
```

**如果不加 lock 和 synchronized，怎样保持原子性**

<img src="https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210406230445.png" alt="image-20210406230445570" style="zoom:67%;" />	

使用原子类，解决原子性问题

<img src="https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210406230556.png" alt="image-20210406230556520" style="zoom:50%;" />	

```java
package com.ouwen.volatile1;

import java.util.concurrent.atomic.AtomicInteger;

/**
 * @author IRVING
 * @create 2021-04-06 22:55
 */
public class Demo2 {

    //volatile不保证原子性
    //使用原子类的Integer
    private volatile static AtomicInteger num = new AtomicInteger();

    public static void add() {
        //num++; //不是一个原子性操作
        num.getAndIncrement();  // AtomicInteger +1 方法 CAS
    }

    public static void main(String[] args) {
        //理论上num结果为2万
        for (int i = 0; i < 20; i++) {
            new Thread(() -> {
                for (int i1 = 0; i1 < 1000; i1++) {
                    add();
                }
            }).start();
        }

        while (Thread.activeCount() > 2) { //main gc
            Thread.yield();
        }

        System.out.println(Thread.currentThread().getName() + "==>" + num);
    }
}
```

这些类的底层都直接和操作系统挂钩！！直接在内存中修改只！Usafe类是一个很特殊的存在！

### 指令重排

什么是指令重排：**你写的程序，计算机并不是按照你写的那样去执行的**

源代码 --> 编译器优化的重排 --> 指令并行也可能会重排 --> 内存也可能会重排 -->执行

==**处理器在进行指令重排的时候，考虑：数据之间的依赖性！**==

可能造成影响的结果：a b x y 四个数都是0

| 线程A | 线程B |
| ----- | ----- |
| x=a   | y=b   |
| b=1   | a=2   |

正常的结果：x=0 y=0  但是可能由于指令重排

| 线程A | 线程B |
| ----- | ----- |
| b=1   | a=2   |
| x=a   | y=b   |

指令重排导致的诡异结果：x=2  y=1

**volatile可以避免指令重排：** 

内存屏障。CPU指令。

作用： 

1、保证特定的操作的执行顺序！

2、可以保证某些变量的内存可见性 （利用这些特性volatile实现了可见性）

<img src="https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210406232313.png" alt="image-20210406232313816" style="zoom:67%;" />		

**Volatile 是可以保持可见性，不能保证原子性，因为内存屏障，可以保证避免指令重排的现象产生！**

## 18、深入理解CAS

### 什么是 CAS？

```java
package com.ouwen.cas;

import java.util.concurrent.atomic.AtomicInteger;

/**
 * @author IRVING
 * @create 2021-04-06 23:52
 */
public class CASDemo {

    //CAS  compareAndSet：比较并交换 compareAndSwap
    public static void main(String[] args) {
        AtomicInteger atomicInteger = new AtomicInteger(2020);

        // 期望、更新
        // public final boolean compareAndSet(int expect, int update)
        // 如果我期望的值达到了，那么就会更新，否则就不更新 CAS 是CPU的并发原语
        System.out.println(atomicInteger.compareAndSet(2020, 2021));
        System.out.println(atomicInteger.get());

        System.out.println(atomicInteger.compareAndSet(2020, 2021));
        System.out.println(atomicInteger.get());
    }
}
```

### Unsafe类

<img src="https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210406235854.png" alt="image-20210406235854081" style="zoom:67%;" />	

<img src="https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210406235912.png" alt="image-20210406235912021" style="zoom:67%;" />	

CAS ： 比较当前工作内存中的值和主内存中的值，如果这个值是期望的，那么则执行操作！如果不是就 一直循环！ 

缺点：

1、 循环会耗时

2、一次性只能保证一个共享变量的原子性 

3、ABA问题

### ABA问题（狸猫换太子）

![image-20210407000535708](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210407000535.png)

```java
package com.ouwen.cas;

import java.util.concurrent.atomic.AtomicInteger;

/**
 * @author IRVING
 * @create 2021-04-06 23:52
 */
public class CASDemo {

    //CAS  compareAndSet：比较并交换 compareAndSwap
    public static void main(String[] args) {
        AtomicInteger atomicInteger = new AtomicInteger(2020);

        // 对于我们写的SQL ：乐观锁！
        // 期望、更新
        // public final boolean compareAndSet(int expect, int update)
        // 如果我期望的值达到了，那么就会更新，否则就不更新 CAS 是CPU的并发原语
        // ==============捣乱的线程==============
        System.out.println(atomicInteger.compareAndSet(2020, 2021));
        System.out.println(atomicInteger.get());

        System.out.println(atomicInteger.compareAndSet(2021, 2020));
        System.out.println(atomicInteger.get());

        // ==============期望的线程==============
        System.out.println(atomicInteger.compareAndSet(2020, 6666));
        System.out.println(atomicInteger.get());
    }
}
```

## 19、原子引用

解决ABA问题，引入原子引用！对应的思想就是我们的乐观锁

带版本号的原子操作！

```java
package com.ouwen.cas;

import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicInteger;
import java.util.concurrent.atomic.AtomicReference;
import java.util.concurrent.atomic.AtomicStampedReference;
import java.util.zip.DeflaterOutputStream;

/**
 * @author IRVING
 * @create 2021-04-07 0:10
 */
public class CASDemo02 {

    public static void main(String[] args) {
        // AtomicStampedReference 注意：如果泛型是一个包装类，注意对象的引用问题
        
        // 正常在业务操作，这里面比较的都是一个个对象
        AtomicStampedReference<Integer> integer = new AtomicStampedReference<>(1, 1);

        //乐观锁的原理
        new Thread(() -> {
            int stamp = integer.getStamp();  //获得版本号
            System.out.println("a1=>"+stamp);

            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            System.out.println(integer.compareAndSet(1, 2, integer.getStamp(), integer.getStamp() + 1));
            System.out.println("a2=>"+integer.getStamp());

            System.out.println(integer.compareAndSet(2, 1, integer.getStamp(), integer.getStamp() + 1));
            System.out.println("a3=>"+integer.getStamp());

        }, "a").start();

        new Thread(() -> {
            int stamp = integer.getStamp();  //获得版本号
            System.out.println("b1=>"+stamp);
            try {
                TimeUnit.SECONDS.sleep(2);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            System.out.println(integer.compareAndSet(1, 6, stamp, stamp + 1));
            System.out.println("b2=>"+integer.getStamp());
        }, "b").start();
    }
}
```

**注意：**

**Integer 使用了对象缓存机制，默认范围是 -128 ~ 127 ，推荐使用静态工厂方法 valueOf 获取对象实 例，而不是 new，因为 valueOf 使用缓存，而 new 一定会创建新的对象分配新的内存空间；**

<img src="https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210407002800.png" alt="image-20210407002800024" style="zoom: 67%;" />	

## 20、各种锁的理解

### 公平锁、非公平锁

公平锁：非常公平，不能够插队，必须先来后到！

非公平锁：非常不公平，可以插队（默认都是非公平）

```java
public ReentrantLock() {
	sync = new NonfairSync();
}

public ReentrantLock(boolean fair) {
	sync = fair ? new FairSync() : new NonfairSync();
}
```

### 可重入锁（递归锁）

<img src="https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210407003348.png" alt="image-20210407003347941" style="zoom:67%;" />	

> synchronized版

```java
package com.ouwen.lock;

/**
 * @author IRVING
 * @create 2021-04-07 0:39
 */
public class Demo01 {

    public static void main(String[] args) {
        Phone phone = new Phone();
        new Thread(()->{
            phone.sms();
        },"A").start();

        new Thread(()->{
            phone.sms();
        },"B").start();
    }
}

class Phone {

    public synchronized void sms() {
        System.out.println(Thread.currentThread().getName() + "sms");
        call(); //这里也有锁
    }

    public synchronized void call() {
        System.out.println(Thread.currentThread().getName() + "call");
    }
}
```

> Lock版

```java
package com.ouwen.lock;

import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

/**
 * @author IRVING
 * @create 2021-04-07 0:39
 */
public class Demo02 {

    public static void main(String[] args) {
        Phone2 phone = new Phone2();
        new Thread(() -> {
            phone.sms();
        }, "A").start();

        new Thread(() -> {
            phone.sms();
        }, "B").start();
    }
}

class Phone2 {

    Lock lock = new ReentrantLock();

    public void sms() {
        lock.lock(); //细节问题：lock.lock()，lock.unlock(); //lock锁必须配对，否则就会死在里面
        lock.lock();
        try {
            System.out.println(Thread.currentThread().getName() + "sms");
            call(); //这里也有锁
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
            lock.unlock();
        }
    }

    public synchronized void call() {
        lock.lock();
        try {
            System.out.println(Thread.currentThread().getName() + "call");
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
}
```

### 自选锁

spinlock

<img src="https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210407004626.png" alt="image-20210407004626556" style="zoom:67%;" />	

**自定义锁：**

```java
package com.ouwen.lock;

import java.util.concurrent.atomic.AtomicReference;

/**
 * 自旋锁
 *
 * @author IRVING
 * @create 2021-04-07 0:52
 */
public class SpinlockDemo {

    AtomicReference<Thread> atomicReference = new AtomicReference<>();

    //加锁
    public void mylock() {
        Thread thread = Thread.currentThread();
        System.out.println(thread.getName() + "==>mylock");

        //自旋锁
        while (!atomicReference.compareAndSet(null, thread)) {

        }
    }

    //解锁
    public void myunlock() {
        Thread thread = Thread.currentThread();
        System.out.println(thread.getName() + "==>myunlock");
        atomicReference.compareAndSet(thread, null);
    }
}
```

**测试代码：**

```java
package com.ouwen.lock;

import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.ReentrantLock;

/**
 * @author IRVING
 * @create 2021-04-07 0:59
 */
public class TestSpinLock {

    public static void main(String[] args) throws InterruptedException {
        //ReentrantLock lock = new ReentrantLock();
        //lock.lock();
        //lock.unlock();

        //地层使用CAS实现自旋锁
        SpinlockDemo lock = new SpinlockDemo();

        new Thread(()->{
            lock.mylock();
            try {
                TimeUnit.SECONDS.sleep(5);
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                lock.myunlock();
            }

        },"T1").start();

        TimeUnit.SECONDS.sleep(1);

        new Thread(()->{

            lock.mylock();
            try {
                TimeUnit.SECONDS.sleep(5);
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                lock.myunlock();
            }

        },"T2").start();
    }
}
```

### 死锁

死锁是什么？

​	<img src="https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210407010904.png" alt="image-20210407010904630" style="zoom:67%;" />

#### 如何解决死锁问题？

1、使用`jps -l`定位进程号

![image-20210407011207227](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210407011207.png)

2、使用`jstack 进程号` 找到死锁问题

![image-20210407011248176](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210407011248.png)

**查看堆栈信息，找到死锁问题！**
