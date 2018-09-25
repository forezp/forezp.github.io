---
layout: post
title:  "Spring Boot教程第20篇：表单提交"
categories: SpringBoot
tags:  SpringBoot
---

* content
{:toc}


这篇文件主要介绍通过springboot 去创建和提交一个表单。

<!--more-->


## 创建工程

涉及了 web，加上spring-boot-starter-web和spring-boot-starter-thymeleaf的起步依赖。

```
<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>


			<dependency>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-starter-thymeleaf</artifactId>
			</dependency>

	</dependencies>

```

## 创建实体

代码清单如下：

```

public class Greeting {

    private long id;
    private String content;

    public long getId() {
        return id;
    }

    public void setId(long id) {
        this.id = id;
    }

    public String getContent() {
        return content;
    }

    public void setContent(String content) {
        this.content = content;
    }

}

```


## 创建Controller

```
@Controller
public class GreetingController {

    @GetMapping("/greeting")
    public String greetingForm(Model model) {
        model.addAttribute("greeting", new Greeting());
        return "greeting";
    }

    @PostMapping("/greeting")
    public String greetingSubmit(@ModelAttribute Greeting greeting) {
        return "result";
    }

}

```

## 页面展示层

src/main/resources/templates/greeting.html

```html
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>Getting Started: Handling Form Submission</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
</head>
<body>
	<h1>Form</h1>
    <form action="#" th:action="@{/greeting}" th:object="${greeting}" method="post">
    	<p>Id: <input type="text" th:field="*{id}" /></p>
        <p>Message: <input type="text" th:field="*{content}" /></p>
        <p><input type="submit" value="Submit" /> <input type="reset" value="Reset" /></p>
    </form>
</body>
</html>

```


src/main/resources/templates/result.html

```html
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>Getting Started: Handling Form Submission</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
</head>
<body>
	<h1>Result</h1>
    <p th:text="'id: ' + ${greeting.id}" />
    <p th:text="'content: ' + ${greeting.content}" />
    <a href="/greeting">Submit another message</a>
</body>
</html>
```

启动工程，访问ttp://localhost:8080/greeting:

![](https://spring.io/guides/gs/handling-form-submission/images/form.png)

点击submit:

![](https://spring.io/guides/gs/handling-form-submission/images/result.png)

## 参考资料

[https://spring.io/guides/gs/handling-form-submission/](https://spring.io/guides/gs/handling-form-submission/)


## 源码下载
[https://github.com/forezp/SpringBootLearning](https://github.com/forezp/SpringBootLearning)

### 优秀文章推荐：

* 更多springboot 教程：[springBoot非官方教程 | 文章汇总](http://blog.csdn.net/forezp/article/details/70341818)
* 更多springcoud 教程：[史上最简单的 SpringCloud 教程 |  文章汇总](http://blog.csdn.net/forezp/article/details/70148833)