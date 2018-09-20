---
layout: post
title:  "javascript 学习6"
categories: javascript
tags:  javascript
---

* content
{:toc}

1.Date 日期对象
日期对象可以储存任意一个日期，并且可以精确到毫秒数（1/1000 秒）。

定义一个时间对象 :

var Udate=new Date(); 
注意:使用关键字new，Date()的首字母必须大写。 

<!--more-->

使 Udate 成为日期对象，并且已有初始值：当前时间(当前电脑系统时间)。

如果要自定义初始值，可以用以下方法：

var d = new Date(2012, 10, 1);  //2012年10月1日
var d = new Date('Oct 1, 2012'); //2012年10月1日
我们最好使用下面介绍的“方法”来严格定义时间。

访问方法语法：“<日期对象>.<方法>”

Date对象中处理时间和日期的常用方法：
![这里写图片描述](http://img.blog.csdn.net/20161013221247570)

```
<!DOCTYPE html>
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<title>获得年份 </title>
<script type="text/javascript">
var mydate=new Date(); 
var myyear=mydate.getFullYear()     ;
document.write("年份:"+myyear);
</script>
</head>
<body>
</body>
</html>
```

2.返回星期方法
getDay() 返回星期，返回的是0-6的数字，0 表示星期天。如果要返回相对应“星期”，通过数组完成，代码如下:

```
<!DOCTYPE HTML>
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<title>获得星期</title>
<script type="text/javascript">
  var mydate=new Date();
  var weekday=["星期日","星期一","星期二","星期三","星期四","星期五","星期六"];
  document.write("今天是：" +weekday[mydate.getDay()] );
</script>
</head>
<body>
</body>
</html>
```

3.
返回/设置时间方法
get/setTime() 返回/设置时间，单位毫秒数，计算从 1970 年 1 月 1 日零时到日期对象所指的日期的毫秒数。

```
<script type="text/javascript">
  var mydate=new Date();
  document.write("当前时间："+mydate+"<br>");
  mydate.setTime(mydate.getTime() + 60 * 60 * 1000);
  document.write("推迟一小时时间：" + mydate);
</script>
```