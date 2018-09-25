---
layout: post
title:  "Spring Boot教程第17篇：上传文件"
categories: SpringBoot
tags:  SpringBoot
---

* content
{:toc}


这篇文章主要介绍，如何在springboot工程作为服务器，去接收通过http 上传的multi-file的文件。

<!--more-->


## 构建工程

为例创建一个springmvc工程你需要spring-boot-starter-thymeleaf和 spring-boot-starter-web的起步依赖。为例能够上传文件在服务器，你需要在web.xml中加入<multipart-config>标签做相关的配置，但在sringboot 工程中，它已经为你自动做了，所以不需要你做任何的配置。


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

## 创建文件上传controller

直接贴代码：

```

@Controller
public class FileUploadController {

    private final StorageService storageService;

    @Autowired
    public FileUploadController(StorageService storageService) {
        this.storageService = storageService;
    }

    @GetMapping("/")
    public String listUploadedFiles(Model model) throws IOException {

        model.addAttribute("files", storageService
                .loadAll()
                .map(path ->
                        MvcUriComponentsBuilder
                                .fromMethodName(FileUploadController.class, "serveFile", path.getFileName().toString())
                                .build().toString())
                .collect(Collectors.toList()));

        return "uploadForm";
    }

    @GetMapping("/files/{filename:.+}")
    @ResponseBody
    public ResponseEntity<Resource> serveFile(@PathVariable String filename) {

        Resource file = storageService.loadAsResource(filename);
        return ResponseEntity
                .ok()
                .header(HttpHeaders.CONTENT_DISPOSITION, "attachment; filename=\""+file.getFilename()+"\"")
                .body(file);
    }

    @PostMapping("/")
    public String handleFileUpload(@RequestParam("file") MultipartFile file,
                                   RedirectAttributes redirectAttributes) {

        storageService.store(file);
        redirectAttributes.addFlashAttribute("message",
                "You successfully uploaded " + file.getOriginalFilename() + "!");

        return "redirect:/";
    }

    @ExceptionHandler(StorageFileNotFoundException.class)
    public ResponseEntity handleStorageFileNotFound(StorageFileNotFoundException exc) {
        return ResponseEntity.notFound().build();
    }

}

```

这个类通过@Controller注解，表明自己上一个Spring mvc的c。每个方法通过
@GetMapping 或者@PostMapping注解表明自己的 http方法。

* GET / 获取已经上传的文件列表
* GET /files/{filename}  下载已经存在于服务器的文件
* POST / 上传文件给服务器

## 创建一个简单的 html模板

为了展示上传文件的过程，我们做一个界面：
在src/main/resources/templates/uploadForm.html

```
<html xmlns:th="http://www.thymeleaf.org">
<body>

	<div th:if="${message}">
		<h2 th:text="${message}"/>
	</div>

	<div>
		<form method="POST" enctype="multipart/form-data" action="/">
			<table>
				<tr><td>File to upload:</td><td><input type="file" name="file" /></td></tr>
				<tr><td></td><td><input type="submit" value="Upload" /></td></tr>
			</table>
		</form>
	</div>

	<div>
		<ul>
			<li th:each="file : ${files}">
				<a th:href="${file}" th:text="${file}" />
			</li>
		</ul>
	</div>

</body>
</html>
```

## 上传文件大小限制

如果需要限制上传文件的大小也很简单，只需要在springboot 工程的src/main/resources/application.properties 加入以下：

```
spring.http.multipart.max-file-size=128KB
spring.http.multipart.max-request-size=128KB

```


## 测试

测试情况如图：


![image.png](http://upload-images.jianshu.io/upload_images/2279594-5f398faeb076e37e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 参考资料

[https://spring.io/guides/gs/uploading-files/](https://spring.io/guides/gs/uploading-files/)

## 源码下载
[https://github.com/forezp/SpringBootLearning](https://github.com/forezp/SpringBootLearning)

### 优秀文章推荐：

* 更多springboot 教程：[springBoot非官方教程 | 文章汇总](http://blog.csdn.net/forezp/article/details/70341818)
* 更多springcoud 教程：[史上最简单的 SpringCloud 教程 |  文章汇总](http://blog.csdn.net/forezp/article/details/70148833)