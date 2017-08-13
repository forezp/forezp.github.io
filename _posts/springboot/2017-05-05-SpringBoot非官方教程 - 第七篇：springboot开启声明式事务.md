---
layout: post
title:  "Spring Boot教程第9篇：声明式事务"
categories: SpringBoot
tags:  SpringBoot
---

* content
{:toc}

springboot开启事务很简单，只需要一个注解@Transactional 就可以了。因为在springboot中已经默认对jpa、jdbc、mybatis开启了事事务，引入它们依赖的时候，事物就默认开启。当然，如果你需要用其他的orm，比如beatlsql，就需要自己配置相关的事物管理器。

<!--more-->

## 准备阶段

以上一篇文章的代码为例子，即springboot整合mybatis，上一篇文章是基于注解来实现mybatis的数据访问层，这篇文章基于xml的来实现，并开启声明式事务。


## 环境依赖

在pom文件中引入mybatis启动依赖：

```
<dependency>
			<groupId>org.mybatis.spring.boot</groupId>
			<artifactId>mybatis-spring-boot-starter</artifactId>
			<version>1.3.0</version>
</dependency>
```

引入mysql 依赖

```
<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
			<scope>runtime</scope>
		</dependency>
		<dependency>
			<groupId>com.alibaba</groupId>
			<artifactId>druid</artifactId>
			<version>1.0.29</version>
		</dependency>

```

## 初始化数据库脚本

```
-- create table `account`
# DROP TABLE `account` IF EXISTS
CREATE TABLE `account` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(20) NOT NULL,
  `money` double DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=4 DEFAULT CHARSET=utf8;
INSERT INTO `account` VALUES ('1', 'aaa', '1000');
INSERT INTO `account` VALUES ('2', 'bbb', '1000');
INSERT INTO `account` VALUES ('3', 'ccc', '1000');

```

## 配置数据源

```
spring.datasource.url=jdbc:mysql://localhost:3306/test
spring.datasource.username=root
spring.datasource.password=123456
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
mybatis.mapper-locations=classpath*:mybatis/*Mapper.xml
mybatis.type-aliases-package=com.forezp.entity

```

通过配置mybatis.mapper-locations来指明mapper的xml文件存放位置，我是放在resources/mybatis文件下的。mybatis.type-aliases-package来指明和数据库映射的实体的所在包。

经过以上步骤，springboot就可以通过mybatis访问数据库来。

## 创建实体类

```

public class Account {
    private int id ;
    private String name ;
    private double money;
    
    getter..
    setter..
    
  }
```

## 数据访问dao 层


接口：

```
public interface AccountMapper2 {
   int update( @Param("money") double money, @Param("id") int  id);
}


```

mapper:

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.forezp.dao.AccountMapper2">


    <update id="update">
        UPDATE account set money=#{money} WHERE id=#{id}
    </update>
</mapper>

```

## service层

```
@Service
public class AccountService2 {

    @Autowired
    AccountMapper2 accountMapper2;

    @Transactional
    public void transfer() throws RuntimeException{
        accountMapper2.update(90,1);//用户1减10块 用户2加10块
        int i=1/0;
        accountMapper2.update(110,2);
    }
}

```
@Transactional，声明事务，并设计一个转账方法，用户1减10块，用户2加10块。在用户1减10 ，之后，抛出异常，即用户2加10块钱不能执行，当加注解@Transactional之后，两个人的钱都没有增减。当不加@Transactional，用户1减了10，用户2没有增加，即没有操作用户2 的数据。可见@Transactional注解开启了事物。

## 结语

springboot 开启事物很简单，只需要加一行注解就可以了，前提你用的是jdbctemplate, jpa, mybatis，这种常见的orm。

源码下载：[https://github.com/forezp/SpringBootLearning](https://github.com/forezp/SpringBootLearning)

## 参考资料

[managing-transactions/](https://spring.io/guides/gs/managing-transactions/)

### 优秀文章推荐：

* 更多springboot 教程：[springBoot非官方教程 | 文章汇总](http://blog.csdn.net/forezp/article/details/70341818)
* 更多springcoud 教程：[史上最简单的 SpringCloud 教程 |  文章汇总](http://blog.csdn.net/forezp/article/details/70148833)