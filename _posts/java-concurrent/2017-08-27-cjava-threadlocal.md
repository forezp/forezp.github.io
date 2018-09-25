---
layout: post
title:  "Java并发编程：线程封闭和ThreadLocal详解"
categories: java并发编程
tags:  java java并发编程 volatile
---

* content
{:toc}



## 什么是线程封闭

![测试](http://fangzhipeng.oss-cn-hangzhou.aliyuncs.com/WechatIMG1.jpeg)
当访问共享变量时，往往需要加锁来保证数据同步。一种避免使用同步的方式就是不共享数据。如果仅在单线程中访问数据，就不需要同步了。这种技术称为线程封闭。在Java语言中，提供了一些类库和机制来维护线程的封闭性，例如局部变量和ThreadLocal类，本文主要深入讲解如何使用ThreadLocal类来保证线程封闭。

<!--more-->

## 理解ThreadLocal类


ThreadLocal类能使线程中的某个值与保存值的对象关联起来，它提供了get、set方法，这些方法为每个使用该变量的线程保存一份独立的副本，因此get总是set当前线程的set最新值。

首先我们来看个例子，这个例子来自于http://www.cnblogs.com/dolphin0520/p/3920407.html

```

public class Test1 {

    ThreadLocal<Long> longLocal = new ThreadLocal<Long>();
    ThreadLocal<String> stringLocal = new ThreadLocal<String>();


    public void set() {
        longLocal.set(Thread.currentThread().getId());
        stringLocal.set(Thread.currentThread().getName());
    }

    public long getLong() {
        return longLocal.get();
    }

    public String getString() {
        return stringLocal.get();
    }
    public static void main(String[] args) throws InterruptedException {
        final Test1 test = new Test1();


        test.set();
        System.out.println(test.getLong());
        System.out.println(test.getString());


        Thread thread1 = new Thread(() -> {
            test.set();
            System.out.println(test.getLong());
            System.out.println(test.getString());
        });
        thread1.start();
        thread1.join();

        System.out.println(test.getLong());
        System.out.println(test.getString());
    }
}

```
运行该程序，代码输出的结果为：

>1
>
> main
> 
>10
>
>Thread-0
>
>1
>
>main
>

从这段代码可以看出在mian线程和thread1线程确实都保存着各自的副本，它们的副本各自不干扰。


## ThreadLocal源码解析

来从源码的角度来解析ThreadLocal这个类，这个类存放在java.lang包，这个类有很多方法。

![image.png](http://upload-images.jianshu.io/upload_images/2279594-903e2b9e3a60cee5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/300)

它内部又个ThreadLocalMap类，主要有set()、get()、setInitialValue 等方法。


首先来看下set方法，获取当前Thread的 map，如果不存在则新建一个并设置值，如果存在设置值，源码如下：

```
public void set(T value) {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
    }
```
跟踪createMap，可以发现它根据Thread创建来一个ThreadLocalMap。

```
  void createMap(Thread t, T firstValue) {
        t.threadLocals = new ThreadLocalMap(this, firstValue);
    }

```
t.threadLocals为当前线程的一个变量，也就是ThreadLocal的数据都是存放在当前线程的threadLocals变量里面的，由此可见用ThreadLocal存放的数据是线程安全的。因为它对于不同的线程来，使用ThreadLocal的set方法都会根据线程判断该线程是否存在它的threadLocals成员变量，如果没有就建一个，有的话就存下数据。

```
ThreadLocal.ThreadLocalMap threadLocals = null;

```

ThreadLocalMap为ThreadLocal的一个内部类，源码如下：


```
 static class ThreadLocalMap {

        static class Entry extends WeakReference<ThreadLocal<?>> {
            /** The value associated with this ThreadLocal. */
            Object value;

            Entry(ThreadLocal<?> k, Object v) {
                super(k);
                value = v;
            }
        }

```

可以看到ThreadLocalMap的Entry继承了WeakReference，并且使用ThreadLocal作为键值。

在使用ThreadLocal的get方法之前一定要先set，要不然会报空指针异常。还有一种方式就是在初始化的时候调用initialValue（）方法赋值。改造下之前的例子，代码如下：

```
public class Test2 {

    ThreadLocal<Long> longLocal = new ThreadLocal<Long>(){

        @Override
        protected Long initialValue() {
            return Thread.currentThread().getId();
        }
    };
    ThreadLocal<String> stringLocal = new ThreadLocal<String>(){
        @Override
        protected String initialValue() {
            return Thread.currentThread().getName();
        }
    };

    public long getLong() {
        return longLocal.get();
    }

    public String getString() {
        return stringLocal.get();
    }

    public static void main(String[] args) throws InterruptedException {
        final Test2 test = new Test2();



        System.out.println(test.getLong());
        System.out.println(test.getString());


        Thread thread1 = new Thread(() -> {
          
            System.out.println(test.getLong());
            System.out.println(test.getString());
        });
        thread1.start();
        thread1.join();

        System.out.println(test.getLong());
        System.out.println(test.getString());
    }
}

```

运行该程序，代码输出的结果为：

>1
>
> main
> 
>10
>
>Thread-0
>
>1
>
>main
>

## ThreadLocal常用的使用场景

通常讲JDBC连接保存在ThreadLocal对象中，每个对象都有属于自己的连接，代码如下：

```
private static ThreadLocal<Connection> connectionHolder
= new ThreadLocal<Connection>() {
    public Connection initialValue() {
       return DriverManager.getConnection(DB_URL);
    }
};
 
public static Connection getConnection() {
    return connectionHolder.get();
}

```

## 参考资料

《Java并发编程实战》

《深入理解JVM》