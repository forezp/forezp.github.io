---
layout: post
title:  "javascript 学习7"
categories: javascript
tags:  javascript
---

* content
{:toc}

1.JavaScript-输出内容（document.write）
```
<script type="text/javascript">
  document.write("I love JavaScript！"); //内容用""括起来，""里的内容直接输出。
</script>
```

```
<script type="text/javascript">
  var mystr="hello world!";
  document.write(mystr);  //直接写变量名，输出变量存储的内容。
</script>
```

<!--more-->

2.JavaScript-警告（alert 消息对话框）

```
<script type="text/javascript">
   var mynum = 30;
   alert("hello!");
   alert(mynum);
</script>

```
3.confirm 消息对话框通常用于允许用户做选择的动作，如：“你对吗？”等。弹出对话框(包括一个确定按钮和一个取消按钮)。
语法:confirm(str);
参数说明:
str：在消息对话框中要显示的文本
返回值: Boolean值

返回值:
当用户点击"确定"按钮时，返回true
当用户点击"取消"按钮时，返回false
```
<script type="text/javascript">
    var mymessage=confirm("你喜欢JavaScript吗?");
    if(mymessage==true)
    {   document.write("很好,加油!");   }
    else
    {  document.write("JS功能强大，要学习噢!");   }
</script>
```

4.JavaScript-提问（prompt 消息对话框）

prompt弹出消息对话框,通常用于询问一些需要与用户交互的信息。弹出消息对话框（包含一个确定按钮、取消按钮与一个文本输入框）。

语法:
prompt(str1, str2);
参数说明：
str1: 要显示在消息对话框中的文本，不可修改
str2：文本框中的内容，可以修改
返回值:
1. 点击确定按钮，文本框中的内容将作为函数返回值
2. 点击取消按钮，将返回null

```
var myname=prompt("请输入你的姓名:");
if(myname!=null)
  {   alert("你好"+myname); }
else
  {  alert("你好 my friend.");  }
```

5.JavaScript-打开新窗口（window.open）
语法：window.open([URL], [窗口名称], [参数字符串])
```
URL：可选参数，在窗口中要显示网页的网址或路径。如果省略这个参数，或者它的值是空字符串，那么窗口就不显示任何文档。
窗口名称：可选参数，被打开窗口的名称。
    1.该名称由字母、数字和下划线字符组成。
    2."_top"、"_blank"、"_self"具有特殊意义的名称。
       _blank：在新窗口显示目标网页
       _self：在当前窗口显示目标网页
       _top：框架网页中在上部窗口中显示目标网页
    3.相同 name 的窗口只能创建一个，要想创建多个窗口则 name 不能相同。
   4.name 不能包含有空格。
参数字符串：可选参数，设置窗口参数，各参数用逗号隔开。
```
例子：
```
<script type="text/javascript"> window.open('http://www.imooc.com','_blank','width=300,height=200,menubar=no,toolbar=no, status=no,scrollbars=yes')
</script>
```
6.JavaScript-关闭窗口（window.close）

window.close();   //关闭本窗口
```
<script type="text/javascript">
   var mywin=window.open('http://www.imooc.com'); //将新打的窗口对象，存储在变量mywin中
   mywin.close();
</script>
```
7.任务

a、新窗口打开时弹出确认框，是否打开
提示: 使用 if 判断确认框是否点击了确定，如点击弹出输入对话框，否则没有任何操作。
b、通过输入对话框，确定打开的网址，默认为 http：//www.imooc.com/
c、打开的窗口要求，宽400像素，高500像素，无菜单栏、无工具栏。
```
<!DOCTYPE html>
<html>
 <head>
  <title> new document </title>  
  <meta http-equiv="Content-Type" content="text/html; charset=gbk"/>   
  <script type="text/javascript">  
    function openmy(){
    var please_confirm=confirm("是否需要打开新窗口")// 新窗口打开时弹出确认框，是否打开
    if(please_confirm==true)
	{
		var text=prompt("请输入网址");
		window.open("http://"+text,'_blank','width=400,high=500,menubar=no,toolbar=no')
		}


    // 通过输入对话框，确定打开的网址，默认为 http：//www.imooc.com/

    //打开的窗口要求，宽400像素，高500像素，无菜单栏、无工具栏。
	}
    
  </script> 
 </head> 
 <body> 
	  <input type="button" value="新窗口打开网站" onclick="openmy()" /> 
 </body>
</html>
```