---
layout: post
title:  "jsp九大内置对象"
categories: j2ee
tags:  j2ee redis
---

* content
{:toc}

jsp内置对象对象是web容器创建的一组对象，不使用new关键词久可以使用的内置对象。
九大内置对象包括以下：

* out  --JspWriter
* request --ServletRequest
* reponse --ServletResponse
* config --ServletConfig
* session --HttpSession
* application --ServlerContext
* page --HttpJspPage
* pageContext --PageContext
* exception --Exception

<!--more-->

### 1、out对象

JspWriter类实例，是向客户端负责输出内容的。
常用方法如下：

* println();
* clear (),如果在flush()之后调用会抛出异常。
* clearBuffer();
* flush();
* isAutoFlush();

举例子：

 ```
 <body>
    <h1>out内置对象</h1>
    <% 
       out.println("<h2>静夜思</h2>");
       out.println("床前明月光<br>");
       out.println("疑是地上霜<br>");
       out.flush();
       //out.clear();//这里会抛出异常。
       out.clearBuffer();//这里不会抛出异常。
       out.println("举头望明月<br>");
       out.println("低头思故乡<br>");
    
    %>
        缓冲区大小：<%=out.getBufferSize() %>byte<br>
        缓冲区剩余大小：<%=out.getRemaining() %>byte<br>
       是否自动清空缓冲区：<%=out.isAutoFlush() %><BR>    
  </body>
 ```
### 2、request对象
 
 客户端的请求被封装在request对象中，通过它可以了解客户端的请求，然后作出响应，request请求具有request请求域。
 常用方法：
 
 * getParameter(String name)
 * getParamterValues(String name)
 * setAttribute(String name,Onject o)
 * getAttribute(string name)
 * getContetType();
 * getProtocol()
 * getServerName();
 
 
 举个例子：用户注册提交数据给request.jsp，在request.jsp页面根据request对象可以获取提交过来的数据。
 
 reg.jsp 注册jsp
 
 ```
 
 <body>
    <h1>用户注册</h1>
    <hr>
    <% 
       int number=-1;
       //说明用户第一次访问页面，计数器对象还未创建
       if(application.getAttribute("counter")==null)
       {
           application.setAttribute("counter", 0);
       }
       number = Integer.parseInt(application.getAttribute("counter").toString());
       number++;
       application.setAttribute("counter", number);
    %>
    <!-- <form name="regForm" action="request.jsp" method="post"> -->
    <form name="regForm" action="response.jsp" method="post">
    <table>
      <tr>
        <td>用户名：</td>
        <td><input type="text" name="username"/></td>
      </tr>
      <tr>
        <td>爱好：</td>
        <td>
           <input type="checkbox" name="favorite" value="read">读书
           <input type="checkbox" name="favorite" value="music">音乐
           <input type="checkbox" name="favorite" value="movie">电影
           <input type="checkbox" name="favorite" value="internet">上网
        </td>
      </tr>
      <tr>
         <td colspan="2"><input type="submit" value="提交"/></td>
      </tr>
    </table>
    </form>
    <br>
    <br>
    <a href="request.jsp?username=李四">测试URL传参数</a>
    
    <br>
    <br>
    <center>
             您是第<%=number %>位访问本页面的用户。
    </center>
  </body>
 
 ```
  
  request.jsp
  
  ```
   <body>
    <h1>request内置对象</h1>
    <% 
       request.setCharacterEncoding("utf-8"); //解决中文乱码问题，无法解决URL传递中文出现的乱码问题。
       request.setAttribute("password", "123456");
    
    %>
        用户名：<%=request.getParameter("username") %><br>   
        爱好 ：<% 
           if(request.getParameterValues("favorite")!=null)
           {
	           String[] favorites = request.getParameterValues("favorite");
	           for(int i=0;i<favorites.length;i++)
	           {
	              out.println(favorites[i]+"  ");
	           }
	        }
        %> <br>
         密码：<%=request.getAttribute("password") %><br> 
         请求体的MIME类型:<%=request.getContentType() %><br>
         协议类型及版本号:  <%=request.getProtocol() %><br>
         服务器主机名 :<%=request.getServerName() %><br>
         服务器端口号：<%=request.getServerPort() %><BR>
         请求文件的长度 ：<%=request.getContentLength() %><BR>
         请求客户端的IP地址：<%=request.getRemoteAddr() %><BR>
         请求的真实路径：<%=request.getRealPath("request.jsp") %><br>
         请求的上下文路径：<%=request.getContextPath() %><BR>         
                 
         
                   
  </body>
  ```
  
### 3、response对象
  
  response对象包含了响应客户端请求的有关信息，它具有页面作用域，该页面的作用域只对该页面有效。常用方法：
  
* getCharacterEncoding()
* setContentType();
* getWriter();该方法打应输出流总是前于 out.println();
* sendRedirect(String location)

*请求重定向和请求转发：*

* 请求重定向：客户端行为:response.sendDirect();两次请求，前一次请求的请求对象不会保存，地址栏的url地址会发生改变
* 请求转发：服务器行为，request.getResuestDispatcher().forward();一次请求，转发后请求对象会保存，地址栏url地址不会变。

### 4、session对象

一些基本概念：

* session表示客户端与服务器的一次会话
* web中session指的是用户在浏览某个网站，是进入网站到关闭浏览器这段时间
* 它是保存在服务器的内存中，不同用户有不同的session
* 它在第一个jsp页面被装载时自动创建，完成会话期管理。

它的的一些常用方法：

* getCreationTime();
* String getId();
* setAttribute(String name,Object o);
* getAttribute(String name);
* String[] getValueNames();
* int getMaxInactivieInterval();单位 秒
* setMaxInactiveInterval();


举个例子：

```

<body>
    <h1>session内置对象</h1>
    <hr>
    <% 
      SimpleDateFormat sdf = new SimpleDateFormat("yyyy年MM月dd日 HH:mm:ss");
      Date d = new Date(session.getCreationTime());
      session.setAttribute("username", "admin"); 
      session.setAttribute("password", "123456");
      session.setAttribute("age", 20);
      
      //设置当前session最大生成期限单位是秒
      //session.setMaxInactiveInterval(10);//10秒钟
      
    %>
    Session创建时间：<%=sdf.format(d)%><br>    
    Session的ID编号：<%=session.getId()%><BR>
         从Session中获取用户名：<%=session.getAttribute("username") %><br>
         
    <a href="session_page2.jsp" target="_blank">跳转到Session_page2.jsp</a>     
        
  </body>
```

*session的生命周期：*

* 创建： 当客户端第一次访问某个页面jsp或者servlet，服务器会创建一个 sessionId,每次客户端向服务器发送请求时，都会将sessionId携带过去，服务器会对sessionId进行校验。
* 活动： 当客户端通过超链接打开新页面属于同一次会话；当浏览页面全部关闭，重新打开属于一次新的会话。
* 销毁：调用sesson.invalidate();session过期，默认是30分钟；服务器重启；

### 5、application对象

* application实现了用户数据共享，可存放全局变量。
* application 开始于服务器的重启，终止于服务器的关闭
* application 是ServletContext实例。

常用方法：

* setAttribute(String ,Object);
* getAttribute(String);
* Enumeration getAttributeNames();
* getServerInfo();返回Jsp 引擎名和版本号


举个例子：

```

<body>
    <h1>application内置对象</h1>
    <% 
       application.setAttribute("city", "北京");
       application.setAttribute("postcode", "10000");
       application.setAttribute("email", "lisi@126.com");
       
    %>
         所在城市是：<%=application.getAttribute("city") %><br>
    application中的属性有：<% 
         Enumeration attributes = application.getAttributeNames();
         while(attributes.hasMoreElements())
         {
            out.println(attributes.nextElement()+"  ");
         }
    %><br>
    JSP(SERVLET)引擎名及版本号:<%=application.getServerInfo() %><br>              
                   
  </body>
```
### 6、page对象

page对象就是指当前jsp页面本身，有点像this指针，它是java.lang.Object类的实例。常用的方法就是Object 类的方法。

* getClass()
* hashCode();
* equals();
* copy();
* clone()
* toString();
* notify();
* notifyAll();
* wait();

### 7、pageContext对象

* pageContext 提供了对jsp页面所有的对象及名字空间的访问
* 它可以取application 某一属性，也可以取session;
* 相当于页面所有功能的集大成者。 

方法：

* getOut()
* geSession();
* getPage();
* getReuest();
* getResponse();
* setAttribute();
* getAttibute();
* getAttributeScope();
* forward();
* include();


### 8、config对象
它是在一个servlet初始化时，jsp页面用它传递信息，比如servlet初始化参数；以及服务器的有关信息。

* ServletContext getServletContext();
* getInitParameter(String);
* Enumeration getInitParameterNames();

### 9、exception对象
即异常对象。如果一个jsp想要用此对象，就必须把isErrorPage 设为true.

```
<%@ page language="java" import="java.util.*" contentType="text/html; charset=utf-8" isErrorPage="true" %>
<%
```

九大内置对象，讲解完毕，感谢大家，后一篇文章会讲述除了jsp的九大内置对象其他内容。