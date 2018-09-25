---
layout: post
title:  "web监听器"
categories: j2ee
tags:  j2ee
---

* content
{:toc}

### 一、简介
Listener是Servlet规范的另一个高级特性，它用于监听java web程序的事件，例如创建、修改、删除session,request,context等，并触发相应的处理事件，这个处理事件是由web容器回掉的。

学过安卓开发的同学一定很熟悉view.setonClickLister();这样的对安卓控件的监听。java web也是这样的 ，根据不同的listner 和不同的event，可以完成相应的处理事件。

<!--more-->

### 二、Listerner的分类
Listerner分为八种，前三种是用于监听对象的创建和销毁，中间三种用于监听对象属性的变化，后两种用于监听Session内对象。

* httpSessionListner: 监听session的创建与销毁，用于收集在线用户信息。
* servletContextListener:监听context的创建与销毁，context代表当前web应用，该listener可用于启动时获取web.xml的初始化参数。
* servletRequestListener:  监听request 的创建与销毁。

* httpSessionAttributeListener 监听session的种属性变化
* ServletContextAttributeListener
* ServletRequestAttributeListener
* HttpSessionBindingListener,监听对象存入或者移除 session
* httpSessionActivationListener,钝化和重新加载 session的监听


### 三、监听session、request、servletContext

直接上代码，下面监听了这三个对象创建销毁。

```
public class ListenerTest implements HttpSessionListener ,ServletContextListener,ServletRequestListener{

	Log log=LogFactory.getLog(getClass());
	public void requestDestroyed(ServletRequestEvent sre) {
		HttpServletRequest request=(HttpServletRequest) sre.getServletRequest();
		long time=System.currentTimeMillis()-(Long)request.getAttribute("time");
		log.info("请求处理时间"+time);
		
	}

	public void requestInitialized(ServletRequestEvent sre) {
		HttpServletRequest request=(HttpServletRequest) sre.getServletRequest();
		String uri=request.getRequestURI();
		uri=request.getQueryString()==null?uri:(uri+"?"+request.getQueryString());
		log.info("ip"+request.getRemoteAddr()+uri);
		request.setAttribute("time", System.currentTimeMillis());
		
	}

	public void contextDestroyed(ServletContextEvent sce) {
		ServletContext servletContext=sce.getServletContext();
		log.info("关闭："+servletContext.getContextPath());
		
	}

	public void contextInitialized(ServletContextEvent sce) {
		ServletContext servletContext=sce.getServletContext();
		log.info("启动："+servletContext.getContextPath());
		
	}

	public void sessionCreated(HttpSessionEvent se) {
		HttpSession session=se.getSession();
		log.info("创建：session:"+session.getId());
		
	}

	public void sessionDestroyed(HttpSessionEvent se) {
		HttpSession session=se.getSession();
		log.info("销毁建：session:"+session.getId());
		
	}

}

```

需要在web.xml中配置：

```
 <listener>
    <listener-class>com.forezp.listener.ListenerTest</listener-class>
 </listener>

```

### 四、监听对象属性的变化

* httpSessionAttributeListener 监听session的种属性变化
* ServletContextAttributeListener
* ServletRequestAttributeListener

以上三种方法用于监听session ,context,request的属性发生变化，例如添加、更新、移除。
下面以session的属性变化为例子：

```
public class SessionAttributeListener  implements HttpSessionAttributeListener{

	Log log=LogFactory.getLog(getClass());
	public void attributeAdded(HttpSessionBindingEvent se) {
		HttpSession httpSession=se.getSession();
		log.info("新建属性："+se.getName()+"值："+se.getValue());
		
	}

	public void attributeRemoved(HttpSessionBindingEvent se) {
		HttpSession httpSession=se.getSession();
		log.info(" 删除属性："+se.getName()+"值："+se.getValue());
		
	}

	public void attributeReplaced(HttpSessionBindingEvent se) {
		HttpSession httpSession=se.getSession();
		log.info(" 修改属性："+se.getName()+"原来的值："+se.getValue()+"新值："+httpSession.getAttribute(se.getName()));
		
	}

}

```
web.xml配置，此处省略。

### 五、监听session内的对象


* HttpSessionBindingListener,当对象被放到session里执行valueBond();当对象被移除，执行valueUnbond();
* httpSessionActivationListener,服务器关闭,会将session的内容保存在硬盘里，这个过程叫钝化；服务器重启，会将session的内容从硬盘中重新加载。钝化时执行sesionWillPassivate(),重新加载sessionDidActivate();

举个例子：

```
public class User implements HttpSessionBindingListener,HttpSessionActivationListener,Serializable {

	private String username;
	private String password;
	
	public void valueBound(HttpSessionBindingEvent httpsessionbindingevent) {
		System.out.println("valueBound Name:"+httpsessionbindingevent.getName());
	}

	public void valueUnbound(HttpSessionBindingEvent httpsessionbindingevent) {
		System.out.println("valueUnbound Name:"+httpsessionbindingevent.getName());
	}

	public String getUsername() {
		return username;
	}

	public void setUsername(String username) {
		this.username = username;
	}

	public String getPassword() {
		return password;
	}

	public void setPassword(String password) {
		this.password = password;
	}

	//钝化
	public void sessionWillPassivate(HttpSessionEvent httpsessionevent) {
		System.out.println("sessionWillPassivate "+httpsessionevent.getSource());
	}
	//活化
	public void sessionDidActivate(HttpSessionEvent httpsessionevent) {
		System.out.println("sessionDidActivate "+httpsessionevent.getSource());
	}

}


```

init.jsp


```

<%@ page language="java" import="java.util.*" pageEncoding="UTF-8"%>
<%
String path = request.getContextPath();
String basePath = request.getScheme()+"://"+request.getServerName()+":"+request.getServerPort()+path+"/";
request.getSession().setAttribute("currentUser", new com.forezp.entity.User());

%>

<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">
<html>
  <head>
    <base href="<%=basePath%>">
    
    <title>My JSP 'init.jsp' starting page</title>
    
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
    这是初始化值的界面
    <button onclick="location.href='<%=request.getContextPath()%>/init.jsp';">Init</button>
    <button onclick="location.href='<%=request.getContextPath()%>/destory.jsp';">Destory</button>
  </body>
</html>

```

destroy.jsp

```
<%@ page language="java" import="java.util.*" pageEncoding="UTF-8"%>
<%
String path = request.getContextPath();
String basePath = request.getScheme()+"://"+request.getServerName()+":"+request.getServerPort()+path+"/";

request.getSession().removeAttribute("currentUser");
%>

<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">
<html>
  <head>
    <base href="<%=basePath%>">
    
    <title>My JSP 'destory.jsp' starting page</title>
    
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
    这是销毁界面
    <button onclick="location.href='<%=request.getContextPath()%>/init.jsp';">Init</button>
    <button onclick="location.href='<%=request.getContextPath()%>/destory.jsp';">Destory</button>
  </body>
</html>

```

当访问init.jsp，再访问destroy.jsp;再访问init,jsp，再关闭服务器，重启；log日志如下：
 
 > valueBound Name:currentUser
 
 > valueUnbound Name:currentUser
 
 >sessionWillPassivate org.apache.catalina.session.StandardSessionFacade@33f3be1
 
 >sessionDidActivate
org.apache.catalina.session.StandardSessionFacade@33f3be1

### 六、显示在线人数：

```

@WebListener
public class MyHttpSessionListener implements HttpSessionListener {
	
	private int userNumber = 0;
	
	@Override
	public void sessionCreated(HttpSessionEvent arg0) {
		userNumber++;
		arg0.getSession().getServletContext().setAttribute("userNumber", userNumber);
	}

	@Override
	public void sessionDestroyed(HttpSessionEvent arg0) {
		userNumber--;
arg0.getSession().getServletContext().setAttribute("userNumber", userNumber);
		
	}

}

```

jsp中显示：

```

<body>
    当前在线用户人数:${userNumber }<br/>
</body>

```

这是一个简答的统计在线人数的方法，如果你需要知道这些人来自哪里，需要配合httpRequestListener配合，也可以实现单登陆，在这里不写代码了。