---
layout: post
title:  "javascript 学习5"
categories: javascript
tags:  javascript
---

* content
{:toc}


1.继续循环continue;
continue的作用是仅仅跳过本次循环，而整个循环体继续执行。
语句结构：



for(初始条件;判断条件;循环后条件值更新)
{
  if(特殊情况)
  { continue; }
 循环代码
}

<!--more-->

2.JavaScript 创建动态页面。事件是可以被 JavaScript 侦测到的行为。 网页中的每个元素都可以产生某些可以触发 JavaScript 函数或程序的事件。

比如说，当用户单击按钮或者提交表单数据时，就发生一个鼠标单击（onclick）事件，需要浏览器做出处理，返回给用户一个结果。

![这里写图片描述](http://img.blog.csdn.net/20161013212433241)

3.鼠标单击事件( onclick ）

onclick是鼠标单击事件，当在网页上单击鼠标时，就会发生该事件。同时onclick事件调用的程序块就会被执行，通常与按钮一起使用。

```
<html>
<head>
   <script type="text/javascript">
      function add2(){
        var numa,numb,sum;
        numa=6;
        numb=8;
        sum=numa+numb;
        document.write("两数和为:"+sum);  }
   </script>
</head>
<body>
   <form>
      <input name="button" type="button" value="点击提交" onclick="add2()" />
   </form>
</body>
</html>
```

4.鼠标经过事件（onmouseover）

鼠标经过事件，当鼠标移到一个对象上时，该对象就触发onmouseover事件，并执行onmouseover事件调用的程序。

```
<!DOCTYPE HTML>
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<title> 鼠标经过事件 </title>
<script type="text/javascript">
    function message(){
      confirm("请输入密码后，再单击确定!"); }
</script>
</head>
<body>
<form>
密码:<input name="password" type="password" >
<input name="确定" type="button" value="确定"  onmouseover="message()"/>
</form>
</body>
</html>
```

5.
鼠标移开事件（onmouseout）
鼠标移开事件，当鼠标移开当前对象时，执行onmouseout调用的程序。
```
<!DOCTYPE HTML>
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<title>鼠标移开事件 </title>
<script type="text/javascript">
  function message(){
    alert("不要移开，点击后进行慕课网!"); }
</script>
</head>
<body>
<form>
  <a href="http://www.imooc.com" onmouseout="message()">点击我</a>
</form>
</body>
</html>
```

6.光标聚焦事件onfocus

当网页中的对象获得聚点时，执行onfocus调用的程序就会被执行。

```
<!DOCTYPE HTML>
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<title> 光标聚焦事件 </title>
  <script type="text/javascript">
    function message(){
	  alert("请选择，您现在的职业！");
	}
  </script>
</head>
<body>
请选择您的职业：<br>
  <form>
    <select name="career" onfocus="message"> 
      <option>学生</option> 
      <option>教师</option> 
      <option>工程师</option> 
      <option>演员</option> 
      <option>会计</option> 
    </select> 
  </form>
</body>
</html>
```

7.失焦事件（onblur）

onblur事件与onfocus是相对事件，当光标离开当前获得聚焦对象的时候，触发onblur事件，同时执行被调用的程序。

```
<!DOCTYPE HTML>
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8">
<title> 失焦事件 </title>
<script type="text/javascript">
  function message(){
    alert("请确定已输入密码后，在移开!"); }
</script>    
</head>
<body>
  <form>
   用户:<input name="username" type="text" value="请输入用户名！" >
   密码:<input name="password" type="text" value="请输入密码！"  onblur="message()">
  </form>
</body>
</html>
```

8.内容选中事件（onselect）

选中事件，当文本框或者文本域中的文字被选中时，触发onselect事件，同时调用的程序就会被执行。

```
<!DOCTYPE HTML>
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8">
<title> 内容选中事件 </title>
<script type="text/javascript">
  function message(){
    alert("您触发了选中事件！"); }
</script>    
</head>
<body>
  <form>
  个人简介：<br>
   <textarea name="summary" cols="60" rows="5"  onselect="message()">请写入个人简介，不少于200字！</textarea>
  </form>
</body>
</html>
```

9.文本框内容改变事件（onchange)

通过改变文本框的内容来触发onchange事件，同时执行被调用的程序。

```
<!DOCTYPE HTML>
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8">
<title> 文本框内容改变事件 </title>
<script type="text/javascript">
  function message(){
    alert("您改变了文本内容！"); }
</script>    
</head>
<body>
  <form>
  个人简介：<br>
   <textarea name="summary" cols="60" rows="5"  onchange="message()">请写入个人简介，不少于200字！</textarea>
  </form>
</body>
</html>
```

10.加载事件（onload）
事件会在页面加载完成后，立即发生，同时执行被调用的程序。
注意：
a. 加载页面时，触发onload事件，事件写在body标签内。
b. 此节的加载页面，可理解为打开一个新页面时。
如下代码,当加载一个新页面时，弹出对话框“加载中，请稍等…”。

```
<!DOCTYPE HTML>
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8">
<title> 加载事件 </title>
<script type="text/javascript">
  function message(){
    alert("加载中，请稍等…"); }
</script>    
</head>
<body onload="message()">
  欢迎学习JavaScript。
</body>
</html>
```

11.卸载事件（onunload）
当用户退出页面时（页面关闭、页面刷新等），触发onUnload事件，同时执行被调用的程序。

注意：不同浏览器对onunload事件支持不同。

```
<!DOCTYPE HTML>
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8">
<title> 卸载事件 </title>
<script type="text/javascript">   
     window.onunload = onunload_message;   
     function onunload_message(){   
        alert("您确定离开该网页吗？");   
    }   
</script>   
</head>
<body>
  欢迎学习JavaScript。
</body>
</html>
```

12.任务
使用JS完成一个简单的计算器功能。实现2个输入框中输入整数后，点击第三个输入框能给出2个整数的加减乘除。

提示：获取元素的值设置和获取方法为：例：赋值：document.getElementById(“id”）.value = 1； 取值：var = document.getElementById(“id”）.value；

```  
<!DOCTYPE html>
<html>
 <head>
  <title> 事件</title>  
  <script type="text/javascript">
   function count(){
       
    //获取第一个输入框的值
    var a=document.getElementById("txt1").value;
	//获取第二个输入框的值
    var b=document.getElementById("txt2").value;
	//获取选择框的值
    var c=document.getElementById("select").value;
	//获取通过下拉框来选择的值来改变加减乘除的运算法则
    
    switch(c){
    case"+":
    var sum=parseInt(a)+parseInt(b);
    break;
    case"-":
    var sum=parseInt(a)-parseInt(b);
    break;
    case"*":
    var sum=parseInt(a)*parseInt(b);
    break;
    case"/":
    var sum=parseInt(a)/parseInt(b);
    break;
    }
    //设置结果输入框的值 
    document.getElementById("fruit").value=sum;
   }
  </script> 
 </head> 
 <body>
   <input type='text' id='txt1' /> 
   <select id='select'>
		<option value='+'>+</option>
		<option value="-">-</option>
		<option value="*">*</option>
		<option value="/">/</option>
   </select>
   <input type='text' id='txt2' /> 
   <input type='button' value=' = ' onclick="count()"/> <!--通过 = 按钮来调用创建的函数，得到结果--> 
   <input type='text' id='fruit' />   
 </body>
</html>
```