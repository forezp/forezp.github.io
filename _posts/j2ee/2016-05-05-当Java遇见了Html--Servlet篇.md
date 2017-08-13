---
layout: post
title:  "servlet"
categories: j2ee
tags:  j2ee
---

* content
{:toc}

###一、什么是servlet

servlet是在服务器上运行的小程序。一个servlet就是一个 java类，并且通过“请求-响应”编程模型来访问的这个驻留在服务器内存里的程序。

<!--more-->

继承关系：

```
servlet(interface)->init(),service(),destroy();
^
genericServlet(abstract class)->与协议无关
^
httpServlet(abstract class)->实现了http协议

```
>servlet 是一个接口，genericServlet是它的一个抽象实现类，但它没有实现任何的协议，httpServlet是genericServlet的子类，实现了http协议，一般我们写servlet需要继承httpServlet。

###二、手工书写第一个servlet程序
1.创建一个web工程，新建一个 servlet包，创建一个HelloServlet类。需要复写doGet()和doPost()方法。

```
public class HelloServlet extends HttpServlet {

	@Override
	protected void doGet(HttpServletRequest req, HttpServletResponse resp)
			throws ServletException, IOException {
		System.out.println("get method invoke");
		PrintWriter out=resp.getWriter();
		out.print("hello get method");
	}

	@Override
	protected void doPost(HttpServletRequest req, HttpServletResponse resp)
			throws ServletException, IOException {
		System.out.println("post method invoke");
		PrintWriter out=resp.getWriter();
		out.print("hi post method");
	}
}

```
在index.jsp上写两个访问servelet的路径方法。其中，< a >标签，是get请求，form表单指明post请求。

```

<%@ page language="java" import="java.util.*" pageEncoding="utf-8"%>
<%
String path = request.getContextPath();
String basePath = request.getScheme()+"://"+request.getServerName()+":"+request.getServerPort()+path+"/";
%>
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">
<html>
  <head>
    <base href="<%=basePath%>">
    
    <title>servelet first</title>
	<meta http-equiv="pragma" content="no-cache">
	<meta http-equiv="cache-control" content="no-cache">
	<meta http-equiv="expires" content="0">    
	<meta http-equiv="keywords" content="keyword1,keyword2,keyword3">
	<meta http-equiv="description" content="This is my page">
	<!--
	<link rel="stylesheet" type="text/css" href="styles.css">
	-->
  </head>
  
  <body>
    <h1>first servlet</h1>
    <a href="servlet/HelloServlet">Get方式请求servlet</a>
    <form action="servlet/HelloServlet" method="post">
    <input type="submit" value="Post方式请求servlet"/>
    </form>
  </body>
</html>
```

配置web.xml，需配置servlet的名字和具体的类名，以及访问路径。

```
<?xml version="1.0" encoding="UTF-8"?>
<web-app version="2.5" xmlns="http://java.sun.com/xml/ns/javaee"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://java.sun.com/xml/ns/javaee 
	http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd">
	<display-name></display-name>
	<welcome-file-list>
		<welcome-file>index.jsp</welcome-file>
	</welcome-file-list>

	<servlet>
		<servlet-name>HelloServlet</servlet-name><!-- servlet的名字 -->
		<servlet-class>servlet.HelloServlet</servlet-class><!-- servlet的具体类型，需要带包名 -->
	</servlet>
	<servlet-mapping>
	<servlet-name>HelloServlet</servlet-name><!-- servlet 名字 -->
	<url-pattern>/servlet/HelloServlet</url-pattern> <!-- servlet访问路径 -->
	</servlet-mapping>
</web-app>
```
部署该工程，运行，访问index.jsp界面，分别点击超链接和form表单，控制台输出类容：

> get method invoke
> 
>post method invoke

这说明，通过页面的访问，点击超链接和form表单提交按钮，分别访问了servelet里的doGet()和doPost().

这里以get请求来说一下 servlet的执行顺序，如图：


![get请求执行流程](http://upload-images.jianshu.io/upload_images/2279594-c1784d7afba86584.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 三、servlet的生命周期

* 初始化阶段，可以是在服务器启动初始化，也可以在第一次调用初始化，生成实例。
* 调用init();
* 调用servicr();判断是哪一种请求，get请求会调用doGet(),post请求调用doPost();
* 在服务器停止的时候，调用destroy();

几个知识点：

* Servlet 启动时自动装载servlet，需要在 web.xml配置，在<servlet></servlet>之间添加如下代码：< loadon-starup >1< /loadon-starup>,数字越小，优先级越高。如果不配置，则第一次访问servlet时装载。
* servlet被加载的时候，会调用init()方法，在整个生命周期中init（） 只会调用一次，装载后，实例贮存在服务器的内存中。


### 四、Servlet获取表单数据
post提交表单，form的method属性设为post, 浏览器即以post方式提交表单内容，与get方式一样，servlet可以通过httpServletRequest对象的getParameter(String para)方式获取param对应的参数值。

如下是jsp提交表单的代码：

```

<body>
    <h1>用户注册</h1>
    <hr>
    <form name="regForm" action="servlet/RegServlet" method="post" >
			  <table border="0" width="800" cellspacing="0" cellpadding="0">
			    <tr>
			    	<td class="lalel">用户名：</td>
			    	<td class="controler"><input type="text" name="username" /></td>
			    </tr>
			    <tr>
			    	<td class="label">密码：</td>
			    	<td class="controler"><input type="password" name="mypassword" ></td>
			    	
			    </tr>
			    <tr>
			    	<td class="label">确认密码：</td>
			    	<td class="controler"><input type="password" name="confirmpass" ></td>
			    	
			    </tr>
			    <tr>
			    	<td class="label">电子邮箱：</td>
			    	<td class="controler"><input type="text" name="email" ></td>
			    	
			    </tr>
			    <tr>
			    	<td class="label">性别：</td>
			    	<td class="controler"><input type="radio" name="gender" checked="checked" value="Male">男<input type="radio" name="gender" value="Female">女</td>
			    	
			    </tr>
			   
			    <tr>
			    	<td class="label">出生日期：</td>
			    	<td class="controler">
			    	  <input name="birthday" type="text" id="control_date" size="10"
                      maxlength="10" onclick="new Calendar().show(this);" readonly="readonly" />
			    	</td>
			    </tr>
			    <tr>
			    	<td class="label">爱好：</td>
			    	<td class="controler">
			    	<input type="checkbox" name="favorite" value="nba"> NBA  
			    	  <input type="checkbox" name="favorite" value="music"> 音乐  
			    	  <input type="checkbox" name="favorite" value="movie"> 电影  
			    	  <input type="checkbox" name="favorite" value="internet"> 上网  
			    	</td>
			    </tr>
			    <tr>
			    	<td class="label">自我介绍：</td>
			    	<td class="controler">
			    		<textarea name="introduce" rows="10" cols="40"></textarea>
			    	</td>
			    </tr>
			    <tr>
			    	<td class="label">接受协议：</td>
			    	<td class="controler">
			    		<input type="checkbox" name="isAccept" value="true">是否接受霸王条款
			    	</td>
			    </tr>
			    <tr>
			    	<td colspan="2" align="center">
			    		<input type="submit" value="注册"/>  
			    	    <input type="reset" value="取消"/>  
			    	</td>
			    </tr>
			  </table>
			</form>
  </body>
```

其中兴趣爱好为多个，即字符串数组，需要使用 httpServletRequest的getParameters(String param)获取.

下面是servlet接受信息：

```

request.setCharacterEncoding("utf-8");

			username = request.getParameter("username");
			mypassword = request.getParameter("mypassword");
			gender = request.getParameter("gender");
			email = request.getParameter("email");
			introduce = request.getParameter("introduce");
			birthday = sdf.parse(request.getParameter("birthday"));
			if(request.getParameterValues("isAccpet")!=null)
			{
			  isAccept = request.getParameter("isAccept");
			}
			else
			{
			  isAccept = "false";
			}
			//用来获取多个复选按钮的值
			favorites = request.getParameterValues("favorite");
		
```

### 五、Servlet之间的跳转
*1.转向（forward)*

转向是通过requestDispatcher对象的forward(httpServletRequest req,HttpServletResponse res)方法来实现的。

```
request.getRequestDispatcher("/servlet/LifeCycleServlet").forward(request,response);
```
其中getRequestDispatcher方法的参数，必须以“／”开头，表示web的根目录，比如要
跳转：　"http://locahost:8080/servlet/servet/LifeCycleServlet", 则参数为"/servlet/LifeCycleServlet".

*2.重定向（redirect)*
通过httpServlet的sendDirect(String location)方法。


### 六、获取初始化参数

1.在 web.xml中配置servlet时，可以配置一些初始化参数，在servlet可以通过servletConfig接口提供的接口获取这些参数。

在web.xml中配置init-param 参数：

```
<servlet>
		
		<servlet-name>HelloServlet</servlet-name><!-- servlet的名字 -->
		<servlet-class>servlet.HelloServlet</servlet-class><!-- servlet的具体类型，需要带包名 -->
		<init-param>
			<param-name>username</param-name>
			<param-value>forezp</param-value>
		</init-param>
	</servlet>
```
在servlet中获取

```

public class HelloServlet extends HttpServlet {
	
	@Override
	public void init() throws ServletException {
		// TODO Auto-generated method stub
		super.init();
		String name=this.getInitParameter("username");
		System.out.println(name);
	}
	
}
```
部署项目，访问helloServlet就可以看见在控制台打印了forezp

2.也可以配置一些全局的参数:context-param.

```
<web-app version="2.5" xmlns="http://java.sun.com/xml/ns/javaee"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://java.sun.com/xml/ns/javaee 
	http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd">
	<display-name></display-name>
	<welcome-file-list>
		<welcome-file>index.jsp</welcome-file>
	</welcome-file-list>


	<context-param>
		<param-name>haha</param-name>
		<param-value>xixi</param-value>
	</context-param>
	
	</web-app>

```

获取方式：

servlet中通过getServletContext()获取servletContext对象，使用ServletContext的getInitParameter()方法获取制定参数名来获取参数。

```
ServletContext servletContext=getServletConfig().getServletContext();
		String str=servletContext.getInitParameter("haha");
		System.out.println(str);

```

### 七、servlet九大内置对象


![Paste_Image.png](http://upload-images.jianshu.io/upload_images/2279594-721dfc4ed65824ba.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

本篇文章是我在学习 servlet的一个总结，参考资料包括《javaweb整合王者归来》，慕课网上的sevlet基础视频。下一篇文章讲讲述jsp的相关内容，尽情期待，感谢大家。