---
layout: post
title:  "javascript 学习9"
categories: javascript
tags:  javascript
---

* content
{:toc}

##  认识DOM 
----
>文档对象模型DOM（Document Object Model）定义访问和处理HTML文档的标准方法。DOM 将HTML文档呈现为带有元素、属性和文本的树结构（节点树）。


<!--more-->

**将HTML代码分解为DOM节点层次图:**

![](http://img.mukewang.com/5375ca7e0001dd8d04830279.jpg)


*HTML文档可以说由节点构成的集合，DOM节点有:*

* 元素节点：上图中<html>、<body>、<p>等都是元素节点，即标签。

* 文本节点:向用户展示的内容中的JavaScript、DOM、CSS等文本。

* 属性节点:元素属性，如<a>标签的链接属性href="http://www.imooc.com"。

----

**节点属性**

![](http://img.mukewang.com/5375c953000117ee05240129.jpg)

**遍历节点树:**

![](http://img.mukewang.com/53f17a6400017d2905230219.jpg)

**DOM操作:**

![](http://img.mukewang.com/538d29da000152db05360278.jpg)

### getElementsByName()方法

**语法：**

```
document.getElementsByName(name)
```

与getElementById() 方法不同的是，通过元素的 name 属性查询元素，而不是通过 id 属性。

**注意：**

* 因为文档中的 name 属性可能不唯一，所有 getElementsByName() 方法返回的是元素的数组，而不是一个元素。

* 和数组类似也有length属性，可以和访问数组一样的方法来访问，从0开始。

----

### getElementsByTagName()方法

返回带有指定标签名的节点对象的集合。返回元素的顺序是它们在文档中的顺序。

*语法：*

```
document.getElementsByTagName(Tagname)
```

*说明：*

* Tagname是标签的名称，如p、a、img等标签名。

* 和数组类似也有length属性，可以和访问数组一样的方法来访问，所以从0开始。


----------

### getAttribute()方法

通过元素节点的属性名称获取属性的值

*语法：*
```
elementNode.getAttribute(name)
```

*说明:*

* elementNode：使用getElementById()、getElementsByTagName()等方法，获取到的元素节点。

* name：要想查询的元素节点的属性名字


----------

### setAttribute()方法

setAttribute() 方法增加一个指定名称和值的新属性，或者把一个现有的属性设定为指定的值。

*语法：*

```
elementNode.setAttribute(name,value)

```

*说明：*

* name: 要设置的属性名。

* value: 要设置的属性值。

##节点属性
在文档对象模型 (DOM) 中，每个节点都是一个对象。DOM 节点有三个重要的属性 ：

* nodeName : 节点的名称

*  nodeValue ：节点的值

*  nodeType ：节点的类型

**一、nodeName 属性:**

节点的名称，是只读的。

* 元素节点的 nodeName 与标签名相同
* 属性节点的 nodeName 是属性的名称
* 文本节点的 nodeName 永远是 #text
* 文档节点的 nodeName 永远是 #document

**二、nodeValue 属性：节点的值**

* 元素节点的 nodeValue 是 undefined 或 null
* 文本节点的 nodeValue 是文本自身
* 属性节点的 nodeValue 是属性的值

**三、nodeType 属性:**

节点的类型，是只读的。以下常用的几种结点类型

元素类似 | 节点类型 
------|-------
元素|1
属性|2
文本|3
注释|8
文档|9


----------


### 访问子结点childNodes

访问选定元素节点下的所有子节点的列表，返回的值可以看作是一个数组，他具有length属性。

*语法：*

```
elementNode.childNodes
```

*注意：*

如果选定的节点没有子节点，则该属性返回不包含节点的 NodeList。


----------


### 访问子结点的第一和最后项

**一、firstChild**
属性返回‘childNodes’数组的第一个子节点。如果选定的节点没有子节点，则该属性返回 NULL。

*语法：*
```
node.firstChild
```
*说明：*
与elementNode.childNodes[0]是同样的效果。 

**二、 lastChild** 

属性返回‘childNodes’数组的最后一个子节点。如果选定的节点没有子节点，则该属性返回 NULL。

*语法：*
```
node.lastChild

```
*说明：*

与elementNode.childNodes[elementNode.childNodes.length-1]是同样的效果。 


----------

### 访问父节点parentNode

获取指定节点的父节点
*语法：*
```
elementNode.parentNode
```
*注意:*
父节点只能有一个。

看看下面的例子,获取 P 节点的父节点，代码如下:
```
<div id="text">
  <p id="con"> parentNode 获取指点节点的父节点</p>
</div> 
<script type="text/javascript">
  var mynode= document.getElementById("con");
  document.write(mynode.parentNode.nodeName);
</script>
```
*运行结果:*
parentNode 获取指点节点的父节点
DIV


----------

### 访问兄弟节点

**1. nextSibling** 属性可返回某个节点之后紧跟的节点（处于同一树层级中）。

*语法：*
```
nodeObject.nextSibling
```
*说明：*如果无此节点，则该属性返回 null。

**2. previousSibling** 属性可返回某个节点之前紧跟的节点（处于同一树层级中）。

*语法：*
```
nodeObject.previousSibling 
```
*说明：*如果无此节点，则该属性返回 null。


----------
### 插入节点appendChild()
在指定节点的最后一个子节点列表之后添加一个新的子节点。

*语法:*
```
appendChild(newnode)
```
*参数:*
newnode：指定追加的节点
 *例子：*
![](http://img.mukewang.com/5398fd020001ad4905890193.jpg)

*运行结果:*

>HTML
JavaScript
This is a new p


### 插入节点insertBefore()
insertBefore() 方法可在已有的子节点前插入一个新的子节点。

*语法:*
```
insertBefore(newnode,node);
```
*参数:*

newnode: 要插入的新节点。
node: 指定此节点前插入节点。

###  删除节点removeChild()

removeChild() 方法从子节点列表中删除某个节点。如删除成功，此方法可返回被删除的节点，如失败，则返回 NULL。

*语法:*
```
nodeObject.removeChild(node)

```
*参数:*
node ：必需，指定需要删除的节点。

我们来看看下面代码，删除子点。

![](http://img.mukewang.com/5399744d000153a306060342.jpg)

*运行结果:*
>HTML


----------


### 删除节点的内容: javascript

替换元素节点replaceChild()
replaceChild 实现子节点(对象)的替换。返回被替换对象的引用。 

*语法：*
```
node.replaceChild (newnode,oldnew )
```

*参数：*

newnode : 必需，用于替换 oldnew 的对象。 
oldnew : 必需，被 newnode 替换的对象。


----------
### 创建元素节点createElement
createElement()方法可创建元素节点。此方法可返回一个 Element 对象。

**语法：**
```
document.createElement(tagName)
```
**参数:**
tagName：字符串值，这个字符串用来指明创建元素的类型。

注意：要与appendChild() 或 insertBefore()方法联合使用，将元素显示在页面中。

我们来创建一个按钮，代码如下：


<script type="text/javascript">
   var body = document.body; 
   var input = document.createElement("input");  
   input.type = "button";  
   input.value = "创建一个按钮";  
   body.appendChild(input);  
 </script> 

 
效果：在HTML文档中，创建一个按钮。


----------

### 创建文本节点createTextNode
createTextNode() 方法创建新的文本节点，返回新创建的 Text 节点。

**语法：**

```
document.createTextNode(data)
```
**参数：**

data : 字符串值，可规定此节点的文本。
我们来创建一个<div>元素并向其中添加一条消息，代码如下

![](http://img.mukewang.com/53951c200001d32d07130554.jpg)