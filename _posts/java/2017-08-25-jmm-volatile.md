---
layout: post
title:  "Java并发编程：JMM和volatile关键字"
categories: java并发编程
tags:  java java并发编程 volatile
---

* content
{:toc}

## Java内存模型

随着计算机的CPU的飞速发展，CPU的运算能力已经远远超出了从主内存（运行内存）中读取的数据的能力，为了解决这个问题，CPU厂商设计出了CPU内置高速缓存区。高速缓存区的加入使得CPU在运算的过程中直接从高速缓存区读取数据，在一定程度上解决了性能的问题。但也引起了另外一个问题，在CPU多核的情况下，每个处理器都有自己的缓存区，数据如何保持一致性。为了保证多核处理器的数据一致性，引入多处理器的数据一致性的协议，这些协议包括MOSI、Synapse、Firely、DragonProtocol等。


![JMM内存模型.png](http://upload-images.jianshu.io/upload_images/2279594-98d10d5cb7acf7a1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

<!--more-->

JVM在执行多线程任务时，共享数据保存在主内存中，每一个线程（执行再不同的处理器）有自己的高速缓存，线程对共享数据进行修改的时候，首先是从主内存拷贝到线程的高速缓存，修改之后，然后从高速缓存再拷贝到主内存。当有多个线程执行这样的操作的时候，会导致共享数据出现不可预期的错误。

举个例子：

> i++;//操作

这个i++操作，线程首先从主内存读取i的值，比如i=0，然后复制到自己的高速缓存区，进行i++操作，最后将操作后的结果从高速缓存区复制到主内存中。如果是两个线程通过操作i++,预期的结果是2。这时结果真的为2吗？答案是否定的。线程1读取主内存的i=0,复制到自己的高速缓存区，这时线程2也读取i=0,复制到自己的高速缓存区，进行i++操作，怎么最终得到的结构为1，而不是2。

为了解决缓存不一致的问题，有两种解决方案：

- 在总线加锁，即同时只有一个线程能执行i++操作（包括读取、修改等）。
- 通过缓存一致性协议


第一种方式就没什么好说的，就是同步代码块或者同步方法。也就只能一个线程能进行对共享数据的读取和修改，其他线程处于线程阻塞状态。
第二种方式就是缓存一致性协议，比如Intel 的MESI协议，它的核心思想就是当某个处理器写变量的数据，如果其他处理器也存在这个变量，会发出信号量通知该处理器高速缓存的数据设置为无效状态。当其他处理需要读取该变量的时候，会让其重新从主内存中读，然后再复制到高速缓存区。

## 编发编程的概念

并发编程的有三个概念，包括原子性、可见性、有序性。

### 原子性

原子性是指，操作为原子性的，要么成功，要么失败，不存在第三种情况。比如：

```
String s="abc";
```

这个复杂操作是原子性的。再比如：

```
int i=0;
i++;
```

i=0这是一个赋值操作，这一步是原子性操作；那么i++是原子性操作吗？当然不是，首先它需要读取i=0，然后需要执行运算，写入i的新值1，它包含了读取和写入两个步骤，所以不是原子性操作。

### 可见性

可见性是指共享数据的时候，一个线程修改了数据，其他线程知道数据被修改，会重新读取最新的主存的数据。
举个例子：

```
i=0;//主内存

i++;//线程1

j=i;//线程2

```
线程1修改了i值，但是没有将i值复制到主内存中，线程2读取i的值，并将i的值赋值给j,我们期望j=1,但是由于线程1修改了，没有来得及复制到主内存中，线程2读取了i,并赋值给j，这时j的值为0。
也就是线程i值被修改，其他线程并不知道。

### 有序性

是指代码执行的有序性，因为代码有可能发生指令重排序（Instruction Reorder）。

Java 语言提供了 volatile 和 synchronized 两个关键字来线程代码操作的有序性，volatile 是因为其本身包含“禁止指令重排序”的语义，synchronized 在单线程中执行代码，无论指令是否重排，最终的执行结果是一致的。

## volatile详解

### volatile关键字作用

被volatile关键字修饰变量，起到了2个作用：
> 1.某个线程修改了被volatile关键字修饰变量是，根据数据一致性的协议，通过信号量，更改其他线程的高速缓存中volatile关键字修饰变量状态为无效状态，其他线程如果需要重写读取该变量会再次从主内存中读取，而不是读取自己的高速缓存中的。
> 
> 2.被volatile关键字修饰变量不会指令重排序。


### volatile能够保证可见性和防止指令重排

在Java并发编程实战一书中有这样


```
public class NoVisibility {
    private static boolean ready;
    private static int a;

    public static void main(String[] args) throws InterruptedException {
        new ReadThread().start();
        Thread.sleep(100);
        a = 32;
        ready = true;
      

    }

    private static class ReadThread extends Thread {
        @Override
        public void run() {
            while (!ready) {
                Thread.yield();
            }
            System.out.println(a);
        }
    }
}

```
在上述代码中，有可能（概率非常小，但是有这种可能性）永远不会打印a的值，因为线程ReadThread读取了主内存的ready为false,主线程虽然更新了ready，但是ReadThread的高速缓存中并没有更新。
另外：

> a = 32;
>
> ready = true;



这两行代码有可能发生指令重排。也就是可以打印出a的值为0。

如果在变量加上volatile关键字，可以防止上述两种不正常的情况的发生。


### volatile不能保证原子性


首先用一段代码测试下，开起了10个线程，这10个线程共享一个变量inc（被volatile修饰），并在每个线程循环1000次对inc进行inc++操作。我们预期的结果是10000.

```
public class VolatileTest {


    public volatile int inc = 0;

    public void increase() {
        inc++;
    }

    public static void main(String[] args) throws InterruptedException {
        final VolatileTest test = new VolatileTest();
        for (int i = 0; i < 10; i++) {
            new Thread(() -> {
                for (int j = 0; j < 1000; j++)
                    test.increase();
            }).start();
        }
        //保证前面的线程都执行完
        Thread.sleep(3000);
        System.out.println(test.inc);
    }

}

```

多次运行main函数，你会发现结果永远都不会为10000，都是小于10000。可能有这样的疑问，volatile保证了共享数据的可见性，线程1修改了inc变量线程2会重新从主内存中重新读，这样就能保证inc++的正确性了啊，可为什么没有得到我们预期的结果呢？

在之前已经讲述过inc++这样的操作不是一个原子性操作，它分为读、加加、写。一种情况，当线程1读取了inc的值，还没有修改，线程2也读取了，线程1修改完了，通知线程2将线程的缓存的 inc的值无效需要重读，可这时它不需要读取inc ，它仍执行写操作，然后赋值给主线程，这时数据就会出现问题。


所以volatile不能保证原子性 。这时需要用锁来保证,在increase方法加上synchronized，重新运行打印的结果为10000 。

```
 public synchronized void increase() {
        inc++;
}

```


### volatile的使用场景

#### 状态标记

volatile最常见的使用场景是状态标记，如下：

```
private volatile boolean asheep ;

//线程1
 
while(!asleep){
    countSheep();
}

//线程2
asheep=true;

```

#### 防止指令重排



```
volatile boolean inited = false;
//线程1:
context = loadContext();  
inited = true;  
//上面两行代码如果不用volatile修饰，可能会发生指令重排，导致报错
 
//线程2:
while(!inited ){
sleep()
}
doSomethingwithconfig(context);

```

## 参考资料

《Java 并发编程实战》

《深入理解JVM》

海子的博客：http://www.cnblogs.com/dolphin0520/p/3920373.html