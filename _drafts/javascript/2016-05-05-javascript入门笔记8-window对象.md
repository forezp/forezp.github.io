---
layout: post
title:  "javascript 学习8"
categories: javascript
tags:  javascript
---

* content
{:toc}

## History 对象
history对象记录了用户曾经浏览过的页面(URL)，并可以实现浏览器前进与后退相似导航的功能。

> 注意:从窗口被打开的那一刻开始记录，每个浏览器窗口、每个标签页乃至每个框架，都有自己的history对象与特定的window对象关联。

<!--more-->

*语法：*

```
window.history.[属性|方法]
```
*History 对象属性*

| 属性 | 描述|
| -------- | -------- |
| length   | 返回浏览器历史列表的URL数量  |

*History 对象方法*

| 方法 | 描述 |
| -------- | -------- |
| back()  | 加载history前一个url  |
| forward()  | 加载history下一个url  |
| go()  | 加载history某一个url  |

例子

```
<script type="text/javascript">
  var HL = window.history.length;
  document.write(HL);
</script>
```

### 返回前一个页面：

```
window.history.back();
```
*或者*

```
window.history.go(-1);
```

### 返回下一个浏览的页面

```
window.history.forward();
```

*或者*

```
window.history.go(1);
```

</br>
## Location对象

location用于获取或设置窗体的URL，并且可以用于解析URL。
*语法*


```
location.[属性|方法]

```
location 对象属性：

![](http://img.mukewang.com/5354b1d00001c4ec06220271.jpg)

location 对象方法:

![](http://img.mukewang.com/5354b1eb00016a2405170126.jpg)

## Navigator对象

*对象属性*

![](http://img.mukewang.com/5354cff70001428b06880190.jpg)

示例：

```
<script type="text/javascript">
   var browser=navigator.appName;
   var b_version=navigator.appVersion;
   document.write("Browser name"+browser);
   document.write("<br>");
   document.write("Browser version"+b_version);
</script>
```

## screen对象
screen对象用于获取用户的屏幕信息。

*语法：*

```
window.screen.属性
```
*d对象属性*
![](http://img.mukewang.com/5354d2810001a47706210213.jpg)

