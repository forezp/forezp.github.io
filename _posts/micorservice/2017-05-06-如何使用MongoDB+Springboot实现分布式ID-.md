---
layout: post
title:  "如何使用MongoDb实现分布式Id"
categories: 微服务
tags:  架构
---

* content
{:toc}


### 一、背景
如何实现分布式id，搜索相关的资料，一般会给出这几种方案：

*   使用数据库自增Id
*   使用reids的incr命令
*   使用UUID
*   Twitter的snowflake算法
*   利用zookeeper生成唯一ID
*   MongoDB的ObjectId


<!--more-->

另外，在我通过爬取知乎用户id发现，知乎的用户id是32位的，初步断定知乎采用的是md5加密，然后全部转换成小写。至于如何爬取知乎用户信息，见我之前分享的文章。本文采取的技术方案采取的是mogoodb的objectId。

### 二.mongodb如何实现分布式ID

MongoDB的ObjectId设计成轻量型的，不同的机器都能用全局唯一的同种方法方便地生成它。MongoDB 从一开始就设计用来作为分布式数据库，处理多个节点是一个核心要求。使其在分片环境中要容易生成得多。

它的格式：
![mongo.png](http://upload-images.jianshu.io/upload_images/2279594-fa59770ee4c176cc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)


* 前4 个字节是从标准纪元开始的时间戳，单位为秒。时间戳，与随后的5 个字节组合起来，提供了秒级别的唯一性。由于时间戳在前，这意味着ObjectId 大致会按照插入的顺序排列。这对于某些方面很有用，如将其作为索引提高效率。这4 个字节也隐含了文档创建的时间。绝大多数客户端类库都会公开一个方法从ObjectId 获取这个信息。 

* 接下来的3 字节是所在主机的唯一标识符。通常是机器主机名的散列值。这样就可以确保不同主机生成不同的ObjectId，不产生冲突。 
为了确保在同一台机器上并发的多个进程产生的ObjectId 是唯一的，接下来的两字节来自产生ObjectId 的进程标识符（PID）。 
* 前9 字节保证了同一秒钟不同机器不同进程产生的ObjectId 是唯一的。
* 后3 字节就是一个自动增加的计数器，确保相同进程同一秒产生的ObjectId 也是不一样的。同一秒钟最多允许每个进程拥有2563（16 777 216）个不同的ObjectId。

### 三、编码

在springboot中引入mongodb:

```

	   <dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>

		<!-- 开启web-->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		

       <!--mongodb -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-mongodb</artifactId>
		</dependency>

```

创建一个实体类：

```
public class Customer {

    @Id
    public String id;

    public String firstName;
    public String lastName;

    public Customer() {}

    public Customer(String firstName, String lastName) {
        this.firstName = firstName;
        this.lastName = lastName;
    }

    @Override
    public String toString() {
        return String.format(
                "Customer[id=%s, firstName='%s', lastName='%s']",
                id, firstName, lastName);
    }


    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public String getFirstName() {
        return firstName;
    }

    public void setFirstName(String firstName) {
        this.firstName = firstName;
    }

    public String getLastName() {
        return lastName;
    }

    public void setLastName(String lastName) {
        this.lastName = lastName;
    }
}


```


创建mongodb 接口类：

```
/**
 * Created by fangzhipeng on 2017/4/1.
 */


public interface CustomerRepository extends MongoRepository<Customer, String> {

    public Customer findByFirstName(String firstName);
    public List<Customer> findByLastName(String lastName);

}

```

测试类：

```

 @Autowired
    CustomerRepository customerRepository;


@Test
public void mongodbIdTest(){
Customer customer=new Customer("lxdxil","dd");
        customer=customerRepository.save(customer);
        logger.info( "mongodbId:"+customer.getId());
}
 
```

### 四、参考资料

[Accessing Data with MongoDB](https://spring.io/guides/gs/accessing-data-mongodb/)

[MongoDB深究之ObjectId](http://www.cnblogs.com/xjk15082/archive/2011/09/18/2180792.html)


[MongoDB 教程](http://www.runoob.com/mongodb/mongodb-databases-documents-collections.html)