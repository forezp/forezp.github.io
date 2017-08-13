---
layout: post
title:  "javascript 学习7"
categories: javascript
tags:  javascript
---

* content
{:toc}

##  计时器
*语法*：

> setInterval(代码，交互时间)

*参数说明：*
 
> 1. 代码：要调用的函数或要执行的代码串。

>2. 交互时间：周期性执行或调用表达式之间的时间间隔，以毫秒计（1s=1000ms）。


<!--more-->

*例子：*

```

<!DOCTYPE HTML>
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8">
<title>定时器</title>
<script type="text/javascript">
  var attime;
  function clock(){
    var time=new Date();          
    attime=  time.getHours()+':'+time.getMinutes()+':'+time.getSeconds();;
    document.getElementById("clock").value = attime;
  }
  var timer = setInterval(clock,1000);
  
</script>
</head>
<body>
<form>
<input type="text" id="clock" size="50"  />
</form>
</body>
</html>

```

##  取消计时器

*语法：*

> clearInterval(id_of_setInterval)

*参数说明：*

> id_of_setInterval：由 setInterval() 返回的 ID 值。

*任务：*

```
<!DOCTYPE HTML>
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8">
<title>计时器</title>
<script type="text/javascript">
   function clock(){
      var time=new Date();               	  
      document.getElementById("clock").value = time;
   }
    var i=setInterval("clock()",100);
     
</script>
</head>
<body>
  <form>
    <input type="text" id="clock" size="50"  />
    <input type="button" value="Stop" onclick="clearInterval(i)"  />
  </form>
</body>
</html>

```

## 延时计时器 setTimeout()

setTimeout()计时器，在载入后延迟指定时间后,去执行一次表达式,仅执行一次。

*语法：*
> setTimeout(代码,延迟时间);

*例子：*

```

<!DOCTYPE HTML>
<html>
<head>
<script type="text/javascript">
function tinfo(){
  var t=setTimeout("alert('Hello!')",5000);
 }
</script>
</head>
<body>
<form>
  <input type="button" value="start" onClick="tinfo()">
</form>
</body>
</html>

```


>要创建一个运行于无穷循环中的计数器，我们需要编写一个函数来调用其自身。在下面的代码，当按钮被点击后，输入域便从0开始计数。


```

<!DOCTYPE HTML>
<html>
<head>
<script type="text/javascript">
var num=0;
function numCount(){
 document.getElementById('txt').value=num;
 num=num+1;
 setTimeout("numCount()",1000);
 }
</script>
</head>
<body>
<form>
<input type="text" id="txt" />
<input type="button" value="Start" onClick="numCount()" />
</form>
</body>
</html>
```

## 取消计时器clearTimeout()

*语法：*
>  clearTimeout(id_of_setTimeout)

*参数说明：*

>id_of_setTimeout：由 setTimeout() 返回的 ID 值。该值标识要取消的延迟执行代码块。

*列子*

```
<!DOCTYPE HTML>
<html>
<head>
<script type="text/javascript">
  var num=0,i;
  function timedCount(){
    document.getElementById('txt').value=num;
    num=num+1;
    i=setTimeout(timedCount,1000);
  }
    setTimeout(timedCount,1000);
  function stopCount(){
    clearTimeout(i);
  }
</script>
</head>
<body>
  <form>
    <input type="text" id="txt">
    <input type="button" value="Stop" onClick="stopCount()">
  </form>
</body>
</html>
```