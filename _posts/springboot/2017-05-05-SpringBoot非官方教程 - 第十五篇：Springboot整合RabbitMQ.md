---
layout: post
title:  "Spring Boot教程第15篇：Rabbitmq"
categories: SpringBoot
tags:  SpringBoot
---

* content
{:toc}


这篇文章带你了解怎么整合RabbitMQ服务器，并且通过它怎么去发送和接收消息。我将构建一个springboot工程，通过RabbitTemplate去通过MessageListenerAdapter去订阅一个POJO类型的消息。

<!--more-->

## 准备工作
* 15min
* IDEA
* maven 3.0

在开始构建项目之前，机器需要安装rabbitmq，你可以去官网下载，http://www.rabbitmq.com/download.html ，如果你是用的Mac（程序员都应该用mac吧），你可以这样下载：

```
brew install rabbitmq

```
安装完成后开启服务器：

```
rabbitmq-server
```

开启服务器成功，你可以看到以下信息：

```
            RabbitMQ 3.1.3. Copyright (C) 2007-2013 VMware, Inc.
##  ##      Licensed under the MPL.  See http://www.rabbitmq.com/
##  ##
##########  Logs: /usr/local/var/log/rabbitmq/rabbit@localhost.log
######  ##        /usr/local/var/log/rabbitmq/rabbit@localhost-sasl.log
##########
            Starting broker... completed with 6 plugins.


```

## 构建工程

构架一个SpringBoot工程，其pom文件依赖加上spring-boot-starter-amqp的起步依赖：

```
<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-amqp</artifactId>
		</dependency>

```

## 创建消息接收者

在任何的消息队列程序中，你需要创建一个消息接收者，用于响应发送的消息。

```

@Component
public class Receiver {

    private CountDownLatch latch = new CountDownLatch(1);

    public void receiveMessage(String message) {
        System.out.println("Received <" + message + ">");
        latch.countDown();
    }

    public CountDownLatch getLatch() {
        return latch;
    }

}

```

消息接收者是一个简单的POJO类，它定义了一个方法去接收消息，当你注册它去接收消息，你可以给它取任何的名字。其中，它有CountDownLatch这样的一个类，它是用于告诉发送者消息已经收到了，你不需要在应用程序中具体实现它，只需要latch.countDown()就行了。

## 创建消息监听，并发送一条消息

在spring程序中，RabbitTemplate提供了发送消息和接收消息的所有方法。你只需简单的配置下就行了：

* 需要一个消息监听容器
* 声明一个quene,一个exchange,并且绑定它们
* 一个组件去发送消息

代码清单如下：

```
package com.forezp;

import com.forezp.message.Receiver;
import org.springframework.amqp.core.Binding;
import org.springframework.amqp.core.BindingBuilder;
import org.springframework.amqp.core.Queue;
import org.springframework.amqp.core.TopicExchange;
import org.springframework.amqp.rabbit.connection.ConnectionFactory;
import org.springframework.amqp.rabbit.listener.SimpleMessageListenerContainer;
import org.springframework.amqp.rabbit.listener.adapter.MessageListenerAdapter;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;


@SpringBootApplication
public class SpringbootRabbitmqApplication {

	 final static String queueName = "spring-boot";

	@Bean
	Queue queue() {
		return new Queue(queueName, false);
	}

	@Bean
	TopicExchange exchange() {
		return new TopicExchange("spring-boot-exchange");
	}

	@Bean
	Binding binding(Queue queue, TopicExchange exchange) {
		return BindingBuilder.bind(queue).to(exchange).with(queueName);
	}

	@Bean
	SimpleMessageListenerContainer container(ConnectionFactory connectionFactory,
											 MessageListenerAdapter listenerAdapter) {
		SimpleMessageListenerContainer container = new SimpleMessageListenerContainer();
		container.setConnectionFactory(connectionFactory);
		container.setQueueNames(queueName);
		container.setMessageListener(listenerAdapter);
		return container;
	}

	@Bean
	MessageListenerAdapter listenerAdapter(Receiver receiver) {
		return new MessageListenerAdapter(receiver, "receiveMessage");
	}


	public static void main(String[] args) {
		SpringApplication.run(SpringbootRabbitmqApplication.class, args);
	}
}


```

创建一个测试方法：


```

@Component
public class Runner implements CommandLineRunner {

    private final RabbitTemplate rabbitTemplate;
    private final Receiver receiver;
    private final ConfigurableApplicationContext context;

    public Runner(Receiver receiver, RabbitTemplate rabbitTemplate,
            ConfigurableApplicationContext context) {
        this.receiver = receiver;
        this.rabbitTemplate = rabbitTemplate;
        this.context = context;
    }

    @Override
    public void run(String... args) throws Exception {
        System.out.println("Sending message...");
        rabbitTemplate.convertAndSend(Application.queueName, "Hello from RabbitMQ!");
        receiver.getLatch().await(10000, TimeUnit.MILLISECONDS);
        context.close();
    }

}

```

启动程序，你会发现控制台打印：

```
Sending message...
Received <Hello from RabbitMQ!>

```

## 总结

恭喜！你刚才已经学会了如何通过spring raabitmq去构建一个消息发送和订阅的程序。 这仅仅是一个好的开始，你可以通过spring-rabbitmq做更多的事，[点击这里](http://docs.spring.io/spring-amqp/reference/html/_introduction.html#quick-tour)。



源码下载：[https://github.com/forezp/SpringBootLearning](https://github.com/forezp/SpringBootLearning)

## 参考资料
[https://spring.io/guides/gs/messaging-rabbitmq/](https://spring.io/guides/gs/messaging-rabbitmq/)


## 优秀文章推荐：

* 更多springboot 教程：[springBoot非官方教程 | 文章汇总](http://blog.csdn.net/forezp/article/details/70341818)
* 更多springcoud 教程：[史上最简单的 SpringCloud 教程 |  文章汇总](http://blog.csdn.net/forezp/article/details/70148833)