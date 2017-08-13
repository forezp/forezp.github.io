
---
layout: post
title:  "java 线程池、信号量"
categories: java
tags:  java
---

* content
{:toc}

当我们需要执行一个异步任务时，通常会创建一个线程并启动它，通常任务执行完，线程会被回收，这的确很方便。但我们有大量的任务需要去执行，高并发的情况下，我们都需要不断的创建线程，创建线程和执行线程任务时非常耗费系统资源的，所以我们需要使用线程池，线程池很好的避免了这种情况，并且能很好的控制线程的执行。

<!--more-->

java中的主要是ThreadPoolExecutor这个类，具体的可以参考下[海子的博客](http://www.cnblogs.com/dolphin0520/p/3932921.html)
```

public class ExcutorService {
	
	
	public static void main(String[] args) {
		 ThreadPoolExecutor executor = new ThreadPoolExecutor(5, 10, 200, TimeUnit.MILLISECONDS,
                 new ArrayBlockingQueue<Runnable>(5));
          
         for(int i=0;i<15;i++){
             MyTask myTask = new MyTask(i);
             executor.execute(myTask);
             System.out.println("线程池中线程数目："+executor.getPoolSize()+"，队列中等待执行的任务数目："+
             executor.getQueue().size()+"，已执行玩别的任务数目："+executor.getCompletedTaskCount());
         }
         executor.shutdown();
	}
	
	
}

```

注意上述代码，如何任务数超过15 会出一场，因为我们在new线程池的时候，就已经指定了个数，即5＋10
```


public class MyTask implements Runnable{
	
	private int taskNum;
    
    public MyTask(int num) {
        this.taskNum = num;
    }
     
    @Override
    public void run() {
        System.out.println("正在执行task "+taskNum);
        try {
            Thread.currentThread().sleep(4000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("task "+taskNum+"执行完毕");
    }

}

```
ThreadPoolExecutor这个类可以很好的控制任务的执行。
当我们需要控制最多5个线程同时进行时，我们需要使用信号量，
acquire()表示需要获取一个许可，当没有许可的时候，线程阻塞，release()表示释放一个许可，下一个阻塞的线程会获取许可，得到执行，通过信号量可以控制现场并发的个数。
```

	public static void main(String[] args) {
		
		Semaphore semaphore=new Semaphore(5);
		ExecutorService executorService=Executors.newFixedThreadPool(5) ;
	
		for( int i=0;i<100;i++){
			final String temp=""+i;
			
			Runnable run =new Runnable() {
	
				@Override
				public void run() {
					try {
						semaphore.acquire();
						
						System.out.println(temp);
						Thread.sleep((long)Math.random()*2000);
						semaphore.release();
					} catch (Exception e) {
						e.printStackTrace();
					}
				
					
				}
			};
			executorService.execute(run);
		}
		
	}
```
