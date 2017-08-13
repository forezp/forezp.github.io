---
layout: post
title:  "javascript 学习1"
categories: javascript
tags:  javascript
---

* content
{:toc}

1.script标签在HTML文件中添加JavaScript代码.

JavaScript代码只能写在HTML文件中吗?当然不是，我们可以把HTML文件和JS代码分开,并单独创建一个JavaScript文件(简称JS文件),其文件后缀通常为.js，然后将JS代码直接写在JS文件中。

<!--more-->

JS文件不能直接运行，需嵌入到HTML文件中执行，我们需在HTML中添加如下代码，就可将JS文件嵌入HTML文件中。

```
<script src="script.js"></script>

```

2.
我们可以将JavaScript代码放在html文件中任何位置，但是我们一般放在网页的head或者body部分。放在head部分;最常用的方式是在页面中head部分放置script元素，浏览器解析head部分就会执行这个代码，然后才解析页面的其余部分。放在body部分;JavaScript代码在网页读取到该语句的时候就会执行.

注意: javascript作为一种脚本语言可以放在html页面中任何位置，但是浏览器解释html时是按先后顺序的，所以前面的script就先被执行。比如进行页面显示初始化的js必须放在head里面，因为初始化都要求提前进行（如给页面body设置css等）；而如果是通过事件调用执行的function那么对位置没什么要求的。

3.
变量名可以任意取名，但要遵循命名规则:
    1.变量必须使用字母、下划线(_)或者美元符($)开始。
  
   2.然后可以使用任意多个英文字母、数字、下划线(_)或者美元符($)组成。
    3.不能使用JavaScript关键词与JavaScript保留字。
注意:

在JS中区分大小写，如变量mychar与myChar是不一样的，表示是两个变量。

变量虽然也可以不声明，直接使用，但不规范，需要先声明，后使用。

4.if(条件)
{ 条件成立时执行的代码 }
else
{ 条件不成立时执行的代码 }

```
<script type="text/javascript">
   var myage = 18;
   if(myage>=18)  //myage>=18是判断条件
   { document.write("你是成年人。");}
   else  //否则年龄小于18
   { document.write("未满18岁，你不是成年人。");}
</script>
```

5.
如何定义一个函数呢？基本语法如下:

function 函数名()
{
     函数代码;
}

```
function add2(){
   var sum = 3 + 2;
   alert(sum);
}
```