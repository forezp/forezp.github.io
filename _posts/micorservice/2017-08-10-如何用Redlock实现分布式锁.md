---
layout: post
title:  "如何使用RedLock实现分布式锁"
categories: SpringCloud
tags:  微服务 分布式锁
---

* content
{:toc}


之前写过一篇文章[《如何在springcloud分布式系统中实现分布式锁？》](http://blog.csdn.net/forezp/article/details/68957681)，由于自己仅仅是阅读了相关的书籍，和查阅了相关的资料，就认为那样的是可行的。那篇文章实现的大概思路是用setNx命令和setEx配合使用。 setNx是一个耗时操作，因为它需要查询这个键是否存在，就算redis的百万的qps，在高并发的场景下，这种操作也是有问题的。关于redis实现分布式锁，redis官方推荐使用redlock。

<!--more-->

## 一、redlock简介

在不同进程需要互斥地访问共享资源时，分布式锁是一种非常有用的技术手段。实现高效的分布式锁有三个属性需要考虑：

* 安全属性：互斥，不管什么时候，只有一个客户端持有锁
* 效率属性A:不会死锁
* 效率属性B：容错，只要大多数redis节点能够正常工作，客户端端都能获取和释放锁。


Redlock是redis官方提出的实现分布式锁管理器的算法。这个算法会比一般的普通方法更加安全可靠。关于这个算法的讨论可以看下[官方文档](https://github.com/antirez/redis-doc/blob/master/topics/distlock.md)。

## 二、怎么用java使用 redlock

在pom文件引入redis和redisson依赖：

```
<!-- redis-->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-redis</artifactId>
		</dependency>
		<!-- redisson-->
		<dependency>
			<groupId>org.redisson</groupId>
			<artifactId>redisson</artifactId>
			<version>3.3.2</version>
		</dependency>

```


AquiredLockWorker接口类，，主要是用于获取锁后需要处理的逻辑：

```
/**
 * Created by fangzhipeng on 2017/4/5.
 * 获取锁后需要处理的逻辑
 */
public interface AquiredLockWorker<T> {
     T invokeAfterLockAquire() throws Exception;
}

```

DistributedLocker 获取锁管理类：

```

/**
 * Created by fangzhipeng on 2017/4/5.
 * 获取锁管理类
 */
public interface DistributedLocker {

     /**
      * 获取锁
      * @param resourceName  锁的名称
      * @param worker 获取锁后的处理类
      * @param <T>
      * @return 处理完具体的业务逻辑要返回的数据
      * @throws UnableToAquireLockException
      * @throws Exception
      */
     <T> T lock(String resourceName, AquiredLockWorker<T> worker) throws UnableToAquireLockException, Exception;

     <T> T lock(String resourceName, AquiredLockWorker<T> worker, int lockTime) throws UnableToAquireLockException, Exception;

}
```

UnableToAquireLockException ，不能获取锁的异常类：

```
/**
 * Created by fangzhipeng on 2017/4/5.
 * 异常类
 */
public class UnableToAquireLockException extends RuntimeException {

    public UnableToAquireLockException() {
    }

    public UnableToAquireLockException(String message) {
        super(message);
    }

    public UnableToAquireLockException(String message, Throwable cause) {
        super(message, cause);
    }
}

```

RedissonConnector 连接类：

```
/**
 * Created by fangzhipeng on 2017/4/5.
 * 获取RedissonClient连接类
 */
@Component
public class RedissonConnector {
    RedissonClient redisson;
    @PostConstruct
    public void init(){
        redisson = Redisson.create();
    }

    public RedissonClient getClient(){
        return redisson;
    }

}

```


RedisLocker 类，实现了DistributedLocker：


```
import org.redisson.api.RLock;
import org.redisson.api.RedissonClient;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;
import java.util.concurrent.TimeUnit;

/**
 * Created by fangzhipeng on 2017/4/5.
 */
@Component
public class RedisLocker  implements DistributedLocker{

    private final static String LOCKER_PREFIX = "lock:";

    @Autowired
    RedissonConnector redissonConnector;
    @Override
    public <T> T lock(String resourceName, AquiredLockWorker<T> worker) throws InterruptedException, UnableToAquireLockException, Exception {

        return lock(resourceName, worker, 100);
    }

    @Override
    public <T> T lock(String resourceName, AquiredLockWorker<T> worker, int lockTime) throws UnableToAquireLockException, Exception {
        RedissonClient redisson= redissonConnector.getClient();
        RLock lock = redisson.getLock(LOCKER_PREFIX + resourceName);
      // Wait for 100 seconds seconds and automatically unlock it after lockTime seconds
        boolean success = lock.tryLock(100, lockTime, TimeUnit.SECONDS);
        if (success) {
            try {
                return worker.invokeAfterLockAquire();
            } finally {
                lock.unlock();
            }
        }
        throw new UnableToAquireLockException();
    }
}

```

测试类：

```
  @Autowired
    RedisLocker distributedLocker;
    @RequestMapping(value = "/redlock")
    public String testRedlock() throws Exception{

        CountDownLatch startSignal = new CountDownLatch(1);
        CountDownLatch doneSignal = new CountDownLatch(5);
        for (int i = 0; i < 5; ++i) { // create and start threads
            new Thread(new Worker(startSignal, doneSignal)).start();
        }
        startSignal.countDown(); // let all threads proceed
        doneSignal.await();
        System.out.println("All processors done. Shutdown connection");
        return "redlock";
    }

     class Worker implements Runnable {
        private final CountDownLatch startSignal;
        private final CountDownLatch doneSignal;

        Worker(CountDownLatch startSignal, CountDownLatch doneSignal) {
            this.startSignal = startSignal;
            this.doneSignal = doneSignal;
        }

        public void run() {
            try {
                startSignal.await();
                distributedLocker.lock("test",new AquiredLockWorker<Object>() {

                    @Override
                    public Object invokeAfterLockAquire() {
                        doTask();
                        return null;
                    }

                });
            }catch (Exception e){

            }
        }

        void doTask() {
            System.out.println(Thread.currentThread().getName() + " start");
            Random random = new Random();
            int _int = random.nextInt(200);
            System.out.println(Thread.currentThread().getName() + " sleep " + _int + "millis");
            try {
                Thread.sleep(_int);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + " end");
            doneSignal.countDown();
        }
    }

```

运行测试类：

> Thread-48 start
Thread-48 sleep 99millis
Thread-48 end
Thread-49 start
Thread-49 sleep 118millis
Thread-49 end
Thread-52 start
Thread-52 sleep 141millis
Thread-52 end
Thread-50 start
Thread-50 sleep 28millis
Thread-50 end
Thread-51 start
Thread-51 sleep 145millis
Thread-51 end



从运行结果上看，在异步任务的情况下，确实是获取锁之后才能运行线程。不管怎么样，这是redis官方推荐的一种方案，可靠性比较高。有什么问题欢迎留言。

## 三、参考资料

[https://github.com/redisson/redisson](https://github.com/redisson/redisson)

[《Redis官方文档》用Redis构建分布式锁](http://ifeve.com/redis-lock/)

[A Look at the Java Distributed In-Memory Data Model (Powered by Redis)](https://dzone.com/articles/java-distributed-in-memory-data-model-powered-by-r)

### 优秀文章推荐：
* [史上最简单的 SpringCloud 教程 | 终章](http://blog.csdn.net/forezp/article/details/70148833)
* [史上最简单的 SpringCloud 教程 | 第一篇: 服务的注册与发现（Eureka）](http://blog.csdn.net/forezp/article/details/69696915)
* [史上最简单的SpringCloud教程 | 第七篇: 高可用的分布式配置中心(Spring Cloud Config)](http://blog.csdn.net/forezp/article/details/70037513)