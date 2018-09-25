---
layout: post
title:  "Spring Boot教程第18篇：定时任务"
categories: SpringBoot
tags:  SpringBoot
---

* content
{:toc}

这篇文章将介绍怎么通过spring去做调度任务。

<!--more-->

## 构建工程

创建一个Springboot工程，在它的程序入口加上@EnableScheduling,开启调度任务。

```
@SpringBootApplication
@EnableScheduling
public class SpringbootSchedulingTasksApplication {

	public static void main(String[] args) {
		SpringApplication.run(SpringbootSchedulingTasksApplication.class, args);
	}
}

```

## 创建定时任务

创建一个定时任务，每过5s在控制台打印当前时间。


```
@Component
public class ScheduledTasks {

    private static final Logger log = LoggerFactory.getLogger(ScheduledTasks.class);

    private static final SimpleDateFormat dateFormat = new SimpleDateFormat("HH:mm:ss");

    @Scheduled(fixedRate = 5000)
    public void reportCurrentTime() {
        log.info("The time is now {}", dateFormat.format(new Date()));
    }
}
```

通过在方法上加@Scheduled注解，表明该方法是一个调度任务。

* @Scheduled(fixedRate = 5000) ：上一次开始执行时间点之后5秒再执行
* @Scheduled(fixedDelay = 5000) ：上一次执行完毕时间点之后5秒再执行
* @Scheduled(initialDelay=1000, fixedRate=5000) ：第一次延迟1秒后执行，之后按fixedRate的规则每5秒执行一次
* @Scheduled(cron="  /5   ") ：通过cron表达式定义规则，什么是cro表达式，自行搜索引擎。

## 测试

启动springboot工程，控制台没过5s就打印出了当前的时间。

>2017-04-29 17:39:37.672  INFO 677 --- [pool-1-thread-1] com.forezp.task.ScheduledTasks           : The time is now 17:39:37
2017-04-29 17:39:42.671  INFO 677 --- [pool-1-thread-1] com.forezp.task.ScheduledTasks           : The time is now 17:39:42
2017-04-29 17:39:47.672  INFO 677 --- [pool-1-thread-1] com.forezp.task.ScheduledTasks           : The time is now 17:39:47
2017-04-29 17:39:52.675  INFO 677 --- [pool-1-thread-1] com.forezp.task.ScheduledTasks           : The time is now 17:39:52


## 总结

在springboot创建定时任务比较简单，只需2步：

* 1.在程序的入口加上@EnableScheduling注解。
* 2.在定时方法上加@Scheduled注解。

## 参考资料

[https://spring.io/guides/gs/scheduling-tasks/](https://spring.io/guides/gs/scheduling-tasks/)

## 源码下载
[https://github.com/forezp/SpringBootLearning](https://github.com/forezp/SpringBootLearning)

### 优秀文章推荐：

* 更多springboot 教程：[springBoot非官方教程 | 文章汇总](http://blog.csdn.net/forezp/article/details/70341818)
* 更多springcoud 教程：[史上最简单的 SpringCloud 教程 |  文章汇总](http://blog.csdn.net/forezp/article/details/70148833)
