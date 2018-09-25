---
layout: post
title:  "jsp详解"
categories: j2ee
tags:  j2ee
---

* content
{:toc}

###一、什么是Jsp
jsp是一种基于文本的程序，全名java server page,其特点是html和java程序共存。执行时jsp会被运行容器编译，编译后的jsp跟servlet一样，因此jsp是另一种形式的servlet。

<!--more-->

### 二、jsp页面组成

jsp 页面包括以下内容：

* 静态内容 
* 指令
* 表达式
* 小脚本
* 声明
* 注释


*1.指令：*

* page指令：  通常位于jsp页面的顶端，同一个页面可以有多个page指令。
* include指令：将一个外部文件嵌入到jsp文件中。
* taglib指令 ：使用标签定义新的自定义标签。

*1.1其中page指令语法：*

```
<%@ page 属性=“属性值”>

```
 属性 |  默认值  
  --------- | --------- 
  language |  java
  import | “”
  
  *1.2* include 指令
  
  ```
  <%@ include file="url" %>
  
  ```  
  *1.3 动作*
  
  * include动作
  
  ```
  <jsp:include page="url" flush="true"/>
  ```
  
  *include 动作和include指令区别*
  
 描述  | include指令  |  include 动作
--- | --- | ---    
  语法 | < % @ include file=""/> | < jsp:include page="url" flush="true"/>    
  发生时间 | 页面转换期间 | 请求期间
  包含内容 | 文件实际内容 |  页面的输出
  转化servlet | 一个servlet |  2个servlet
  编译时间 | 较慢 | 较快
  执行时间 | 稍快 |较慢--每次资源必须被编译
  
  *forward动作*
  
  ```
  <jsp:forward page="url"/>
  
  ==
  
  request.getRequestDispatcher("/url").forward(res,resp);
  
  ```
   *param动作*
   
   ```
   <jsp:param name="参数名" value="参数值"/>
   常常与<jsp:forward>一起使用
   
   ```
   
   例子：
   
   ```
   <jsp:forward page="user.jsp">
   
      <jsp:param name="email"  value="1233@154.com"/>
   </jsp:forward>
   
   ```
  
  *2.jsp注释* 
  
  * html注释
  
  ```
  <!-- html注释 -->//客户端可见
  ```
  * jsp 注释 
  
  ```
  <%-- jsp注释 --%>//客户端不可见
  ```
  
  * jsp 脚本注释  //客户端不可见


  ```
  //单行注射
  ／** 多行注释*／
  ```

*3.jsp脚本* 

在jsp页面中执行的java代码，语法：

```
<%   java 代码 %>
```
*4.jsp声明* 
在jsp页面定义变量或者方法，语法

```
<%! java 代码  %>
```
举例：

```
<%! 
 String s="adele";
 int add(int x,int y){
    return x+y;
 }
%>

```
*5.jsp表达式* 
在jsp页面执行的表达式，语法：

```
<% =表达式 %>// 表达式不以分号结尾

```

举例：

```
<%! 
 String s="adele";

%>
<h2>  hello,<%=s %> </h2>

```

### 三、jsp生命周期

![CC36B22A-503E-4C29-98BB-3B58038C140E.png](http://upload-images.jianshu.io/upload_images/2279594-70294c4b278ed9b9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

jspService()是用来处理客户端请求的，对于每一个请求，服务器会创建一个新的线程来处理该请求。以多线程方式执行大大降低对系统的资源需求，提高系统的并发量和缩短了响应时间，servlet是常驻在服务器内存中。

它同servlet 一样，jsp 实例初始化和销毁也会调用sevlet的init() 和destroy();
另外jsp还有自己的初始化方法_jspInit();_jspDestroy();

```
<%@ page language="java" contentType="text/html";charset="utf-8">

<%! 
public void _jspInit(){
}
public void _jspDestroy(){
}
%>
```

### 四、javaben的使用

*动作元素：*
动作元素为请求处理阶段提供信息。

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/2279594-cd9657ce138ebed4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


在jsp页面使用javaben

* 像普通的java类一样，创建javabean;
*  在jsp使用动作标签来使用 javaben

相关标签如下：

```
<jsp:useBwan id="" class="" scope="" />

<jsp:setProperty name="javabean 是例"  property="*"/>(跟表单关联)

<jsp:setProperty name="javabean 是例"  property="javaben 属性名"/>(跟表单关联)

<jsp:setProperty name="javabean 是例"  property="javaben 属性名"  value=""/>(手动设置)


<jsp:setProperty name="javabean 是例"  property="javaben 属性名"  param="request对象参数"/>(跟request参数关联)


<jsp:getProperty name="" property=""/>
```

举个例子：
首先用户 在login.jsp提交表单，然后用户在dologin.jsp 根据动作标签获取参数。


login.jsp

```

 <form name="loginForm" action="dologin.jsp?mypass=999999" method="post">
      <table>
        <tr>
          <td>用户名：</td>
          <td><input type="text" name="username" value=""/></td>
        </tr>
        <tr>
          <td>密码：</td>
          <td><input type="password" name="password" value=""/></td>
        </tr>
        <tr>
          <td colspan="2" align="center"><input type="submit" value="登录"/></td>
          
        </tr>
      </table>
    </form>
```


dologin.jsp

```

<body>
    <jsp:useBean id="myUsers" class="com.po.Users" scope="page"/>
    <h1>setProperty动作元素</h1>
    <hr>
   <!--根据表单自动匹配所有的属性 -->
   <%-- 
   <jsp:setProperty name="myUsers" property="*"/>  
   --%>
   <!--根据表单匹配所有部分的属性 -->
   <%-- 
   <jsp:setProperty name="myUsers" property="username"/>  
   --%>
   <!--根表单无关，通过手工赋值给属性 -->
   <%-- 
   <jsp:setProperty name="myUsers" property="username" value="lisi"/>
   <jsp:setProperty name="myUsers" property="password" value="888888"/>
   --%>
   <!--通过URL传参数给属性赋值 -->
   <jsp:setProperty name="myUsers" property="username"/>
   <jsp:setProperty name="myUsers" property="password" param="mypass"/>
   <!-- 使用传统的表达式方式来获取用户名和密码 -->
   <%--     
       用户名：<%=myUsers.getUsername() %><br>
       密码：<%=myUsers.getPassword() %><br> 
   --%>
   <!-- 使用getProperty方式来获取用户名和密码 -->
      用户名：<jsp:getProperty name="myUsers" property="username"/> <br>
      密码：<jsp:getProperty name="myUsers" property="password"/><br>
   <br>
   <br>
      <a href="testScope.jsp">测试javabean的四个作用域范围</a>
      <% 
         request.getRequestDispatcher("testScope.jsp").forward(request, response);
      %>
  </body>

```

*javaben 四大作用域*

* page ，仅当前页面有效
* request  ,通过httpRequest.getAttribute()获取jvabean对象
* session ,通过httpSession.getAttribute() 获取javabean对象
* application，通过application.getAttribute方法获取javabean 对象。


### 五、cookie

*1.概述：*

由于http协议的无状态，无法保存用户的状态，所以需要用session和cookie.

cookie 是web服务器保存在客户端的一系列文本信息。它的作用时记录一些用户的行为，简化登陆，但是容易泄露用户信息。

*2.jsp创建和使用cookie*

* 创建cookie

```
Cookie cookie=new Cookie(String ,Object);
```

* 写入cookie

```
response.addCookie(cookie);
```

* 读取 cookie

```
Cookie[] cookies=request.getCookies();

```

*3.cookie的常用方法*

* setMaxAge();
* setValue();
* getName();
* getValue();
* getMaxAge();

举个列子： 使用cookie记住用户登陆的账号密码；

登陆界面：

```

<body>
    <h1>用户登录</h1>
    <hr>
    <% 
      request.setCharacterEncoding("utf-8");
      String username="";
      String password = "";
      Cookie[] cookies = request.getCookies();
      if(cookies!=null&&cookies.length>0)
      {
           for(Cookie c:cookies)
           {
              if(c.getName().equals("username"))
              {
                   username =  URLDecoder.decode(c.getValue(),"utf-8");
              }
              if(c.getName().equals("password"))
              {
                   password =  URLDecoder.decode(c.getValue(),"utf-8");
              }
           }
      }
    %>
    <form name="loginForm" action="dologin.jsp" method="post">
       <table>
         <tr>
           <td>用户名：</td>
           <td><input type="text" name="username" value="<%=username %>"/></td>
         </tr>
         <tr>
           <td>密码：</td>
           <td><input type="password" name="password" value="<%=password %>" /></td>
         </tr>
         <tr>
           <td colspan="2"><input type="checkbox" name="isUseCookie" checked="checked"/>十天内记住我的登录状态</td>
         </tr>
         <tr>
           <td colspan="2" align="center"><input type="submit" value="登录"/><input type="reset" value="取消"/></td>
         </tr>
       </table>
    </form>
  </body>

```

处理登陆逻辑的jsp

```
 <body>
    <h1>登录成功</h1>
    <hr>
    <br>
    <br>
    <br>
    <% 
       request.setCharacterEncoding("utf-8");
       //首先判断用户是否选择了记住登录状态
       String[] isUseCookies = request.getParameterValues("isUseCookie");
       if(isUseCookies!=null&&isUseCookies.length>0)
       {
          //把用户名和密码保存在Cookie对象里面
          String username = URLEncoder.encode(request.getParameter("username"),"utf-8");
          //使用URLEncoder解决无法在Cookie当中保存中文字符串问题
          String password = URLEncoder.encode(request.getParameter("password"),"utf-8");
          
          Cookie usernameCookie = new Cookie("username",username);
          Cookie passwordCookie = new Cookie("password",password);
          usernameCookie.setMaxAge(864000);
          passwordCookie.setMaxAge(864000);//设置最大生存期限为10天
          response.addCookie(usernameCookie);
          response.addCookie(passwordCookie);
       }
       else
       {
          Cookie[] cookies = request.getCookies();
          if(cookies!=null&&cookies.length>0)
          {
             for(Cookie c:cookies)
             {
                if(c.getName().equals("username")||c.getName().equals("password"))
                {
                    c.setMaxAge(0); //设置Cookie失效
                    response.addCookie(c); //重新保存。
                }
             }
          }
       }
    %>
    <a href="users.jsp" target="_blank">查看用户信息</a>
    
  </body>


```


使用cookie获取用户信息：

```
 <body>
    <h1>用户信息</h1>
    <hr>
    <% 
      request.setCharacterEncoding("utf-8");
      String username="";
      String password = "";
      Cookie[] cookies = request.getCookies();
      if(cookies!=null&&cookies.length>0)
      {
           for(Cookie c:cookies)
           {
              if(c.getName().equals("username"))
              {
                   username = URLDecoder.decode(c.getValue(),"utf-8");
              }
              if(c.getName().equals("password"))
              {
                   password = URLDecoder.decode(c.getValue(),"utf-8");
              }
           }
      }
    %>
    <BR>
    <BR>
    <BR>
         用户名：<%=username %><br>
         密码：<%=password %><br>
  </body>

```

*3.cookie和 session的区别*

session |cookie 
--- | ---
在服务端保存信息 | 在客户端保存信息
保存的 object类型 | 保存的是 string 类型
随会话结束，销毁数据 | 可以长期保存在客户端中
重要信息 | 不重要信息