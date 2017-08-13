---
layout: post
title:  "javascript 学习3"
categories: javascript
tags:  javascript
---

* content
{:toc}

1.通过ID获取元素
 document.getElementById(“id”) 
```
<!DOCTYPE HTML>
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<title>document.getElementById</title>
</head>
<body>
<p id="con">JavaScript</p>
<script type="text/javascript">
  var mychar=   document.getElementById("con")        ;
  document.write("结果:"+mychar); //输出获取的P标签。 
</script>
</body>
</html>
```

<!--more-->

2.
innerHTML 属性用于获取或替换 HTML 元素的内容。
语法：Object.innerHTML

1.Object是获取的元素对象，如通过document.getElementById("ID")获取的元素。

2.注意书写，innerHTML区分大小写。

```
<!DOCTYPE HTML>
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<title>innerHTML</title>
</head>
<body>
<h2 id="con">javascript</H2>
<p> JavaScript是一种基于对象、事件驱动的简单脚本语言，嵌入在HTML文档中，由浏览器负责解释和执行，在网页上产生动态的显示效果并实现与用户交互功能。</p>
<script type="text/javascript">
  var mychar=   document.getElementById("con")        ;
  document.write("原标题:"+mychar.innerHTML+"<br>"); //输出原h2标签内容
  mychar.innerHTML="ddd";
  document.write("修改后的标题:"+mychar.innerHTML); //输出修改后h2标签内容
</script>
</body>
</html>
```

3.改变 HTML 样式
HTML DOM 允许 JavaScript 改变 HTML 元素的样式。如何改变 HTML 元素的样式呢？

语法:
Object.style.property=new style;

注意:Object是获取的元素对象，如通过document.getElementById("id")获取的元素。
```
<!DOCTYPE HTML>
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<title>style样式</title>
</head>
<body>
  <h2 id="con">I love JavaScript</H2>
  <p> JavaScript使网页显示动态效果并实现与用户交互功能。</p>
  <script type="text/javascript">
    var mychar= document.getElementById("con");
    mychar.style.color="red";
    mychar.style.backgroundColor="#CCC";
    mychar.style.width="300";
  </script>
</body>
</html>
```
4.显示和隐藏（display属性）
语法：
Object.style.display = value

value值：
none:隐藏
block:显示
```
<!DOCTYPE HTML>
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=gb2312">
<title>display</title>
    <script type="text/javascript"> 
        function hidetext()  
		{  
		var mychar = document.getElementById("con");
        mychar.style.display="none";
		}  
		function showtext()  
		{  
		var mychar = document.getElementById("con");
        mychar.style.display="block";
        
		}
    </script> 
</head> 
<body>  
    <h1>JavaScript</h1>  
    <p id="con">做为一个Web开发师来说，如果你想提供漂亮的网页、令用户满意的上网体验，JavaScript是必不可少的工具。</p> 
    <form>
       <input type="button" onclick="hidetext()" value="隐藏内容" /> 
       <input type="button" onclick="showtext()" value="显示内容" /> 
    </form>
</body> 
</html>
```
5.控制类名（className 属性）
语法：
object.className = classname

作用:
1.获取元素的class 属性
2. 为网页内的某个元素指定一个css样式来更改该元素的外观
例子：

```

<!DOCTYPE HTML>
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=gb2312">
<title>className属性</title>
<style>
    body{ font-size:16px;}
    .one{
		border:1px solid #eee;
		width:230px;
		height:50px;
		background:#ccc;
		color:red;
    }
	.two{
		border:1px solid #ccc;
		width:230px;
		height:50px;
		background:#9CF;
		color:blue;
	}
	</style>
</head>
<body>
    <p id="p1" > JavaScript使网页显示动态效果并实现与用户交互功能。</p>
    <input type="button" value="添加样式" onclick="add()"/>
	<p id="p2" class="one">JavaScript使网页显示动态效果并实现与用户交互功能。</p>
    <input type="button" value="更改外观" onclick="modify()"/>

	<script type="text/javascript">
	   function add(){
	      var p1 = document.getElementById("p1");
	      p1.className="one";
	   }
	   function modify(){
	      var p2 = document.getElementById("p2");
	      p2.className="two";
	   }
	</script>
</body>
</html>
```

6.任务
一、定义"改变颜色"的函数

提示:
obj.style.color
obj.style.backgroundColor 
二、定义"改变宽高"的函数

提示:
obj.style.width
obj.style.height 
三、定义"隐藏内容"的函数

提示:
obj.style.display="none";
四、定义"显示内容"的函数

提示:
obj.style.display="block";
五、定义"取消设置"的函数

提示: 
使用confirm()确定框，来确认是否取消设置。
如是将以上所有的设置恢复原始值,否则不做操作。
六、当点击相应按钮，执行相应操作，为按钮添加相应事件

答案：
```
<!DOCTYPE HTML>
<html>
<head>
<meta http-equiv="Content-Type" Content="text/html; charset=utf-8" />
<title>javascript</title>
<style type="text/css">
body{font-size:12px;}
#txt{
    height:400px;
    width:600px;
	border:#333 solid 1px;
	padding:5px;}
p{
	line-height:18px;
	text-indent:2em;}
</style>
</head>
<body>
  <h2 id="con">JavaScript课程</H2>
  <div id="txt"> 
     <h5>JavaScript为网页添加动态效果并实现与用户交互的功能。</h5>
        <p>1. JavaScript入门篇，让不懂JS的你，快速了解JS。</p>
        <p>2. JavaScript进阶篇，让你掌握JS的基础语法、函数、数组、事件、内置对象、BOM浏览器、DOM操作。</p>
        <p>3. 学完以上两门基础课后，在深入学习JavaScript的变量作用域、事件、对象、运动、cookie、正则表达式、ajax等课程。</p>
  </div>
  <form>
  <!--当点击相应按钮，执行相应操作，为按钮添加相应事件-->
    <input type="button" value="改变颜色" onclick="clo()">  
    <input type="button" value="改变宽高" onclick="wid()">
    <input type="button" value="隐藏内容" onclick="hid()">
    <input type="button" value="显示内容" onclick="blo()">
    <input type="button" value="取消设置" onclick="res()">
  </form>
  <script type="text/javascript">
//定义"改变颜色"的函数
    function clo(){
        var p = document.getElementById("txt");
        p.style.color="red";   
    }
//定义"改变宽高"的函数
    function wid(){
        var p = document.getElementById("txt");
        p.style.width="300px";
        p.style.height="300px";
    }
//定义"隐藏内容"的函数
    function hid(){
        var p = document.getElementById("txt");
        p.style.display = "none";
    }
//定义"显示内容"的函数
    function blo(){
        var p = document.getElementById("txt");
        p.style.display = "block";
    }
//定义"取消设置"的函数
    function res(){
        var p1 = confirm("是否取消");
        var p = document.getElementById("txt");
        if(p1==true){
            p.removeAttribute('style');
        }
    }
  </script>
</body>
</html>
```