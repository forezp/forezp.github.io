---
layout: post
title:  "javascript 学习4"
categories: javascript
tags:  javascript
---

* content
{:toc}

1.数组
var arr=new Array();
var myarray= new Array(8); //创建数组，存储8个数据。 
注意：
1.创建的新数组是空数组，没有值，如输出，则显示undefined。
2.虽然创建数组时，指定了长度，但实际上数组都是变长的，也就是说即使指定了长度为8，仍然可以将元素存储在规定长度以外。

<!--more-->

a.
```
var myarray = new Array(66,80,90,77,59);//创建数组同时赋值
```
b.
```
var myarray = [66,80,90,77,59];//直接输入一个数组（称 “字面量数组”）
```


c.
```
<script type="text/javascript">
 var myarr=new Array(); //定义数组
 myarr[0]=80; 
 myarr[1]=60;
 myarr[2]=99;
 document.write("第一个人的成绩是:"+myarr[0]);
 document.write("第二个人的成绩是:"+myarr[1]);
 document.write("第三个人的成绩是:"+myarr[2]);
</script>
```
2.了解成员数量(数组属性length)
myarray.length; //获得数组myarray的长度

JavaScript数组的length属性是可变的，这一点需要特别注意。
```
arr.length=10; //增大数组的长度
document.write(arr.length); //数组长度已经变为10

var arr=[98,76,54,56,76]; // 包含5个数值的数组
document.write(arr.length); //显示数组的长度5
arr[15]=34;  //增加元素，使用索引为15,赋值为34
alert(arr.length); //显示数组的长度16
```

3.二维数组，我们看成一组盒子，不过每个盒子里还可以放多个盒子。

二维数组的表示: myarray[ ][ ]

```
1. 二维数组的定义方法一

var myarr=new Array();  //先声明一维 
for(var i=0;i<2;i++){   //一维长度为2
   myarr[i]=new Array();  //再声明二维 
   for(var j=0;j<3;j++){   //二维长度为3
   myarr[i][j]=i+j;   // 赋值，每个数组元素的值为i+j
   }
 }


2. 二维数组的定义方法二

var Myarr = [[0 , 1 , 2 ],[1 , 2 , 3]]

3. 赋值

myarr[0][1]=5; //将5的值传入到数组中，覆盖原有值。

说明: myarr[0][1] ,0 表示表的行，1表示表的列。

 
```


