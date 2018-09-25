---
layout: post
title:  "Openresrt最佳案例 | 第2篇：Lua入门"
categories: Openresty 
tags:  Openresty Nginx
---

* content
{:toc}


> Lua 是一种轻量小巧的脚本语言，用标准C语言编写并以源代码形式开放， 其设计目的是为了嵌入应用程序中，从而为应用程序提供灵活的扩展和定制功能。


> Lua 是巴西里约热内卢天主教大学（Pontifical Catholic University of Rio de Janeiro）里的一个研究小组，由Roberto Ierusalimschy、Waldemar Celes 和 Luiz Henrique de Figueiredo所组成并于1993年开发。
>                ---摘抄 http://www.runoob.com/lua/lua-tutorial.html

<!--more-->

## 环境搭建

注意： 在上一篇文章中，OpenResty已经有了Lua的环境，这里安装的是单独的Lua环境，用于学习和开发Lua。大多数的电脑是Windowds版本的电脑，Windows版本下载地址http://luaforge.net/projects/luaforwindows/。


Linux和Mac电脑下载地址：http://luajit.org/download.html，安装命令如下：

```
wget http://luajit.org/download/LuaJIT-2.1.0-beta1.tar.gz

tar -xvf LuaJIT-2.1.0-beta1.tar.gz
cd LuaJIT-2.1.0-beta1
make
sudo make install

```

使用IDEA开发的同学，可以通过安装插件的形式来集成Lua的环境，插件名为EmmyLua，安装插件后，在Idea的右侧栏就会出现Lua的图标，点击图标，就会出现运行Lua代码的窗口。建议使用该插件，可以免去安装Lua环境的麻烦。


## 第一个Lua程序

安装好环境后，我采用EmmyLua插件的形式，对Lua的入门语法进行一个简单的讲解。
打开EmmyLua的终端，在终端上输入：

```
print("hi you")

```
按ctrl+enter，终端显示：

> hi you

## Lua基本数据类型

lua的基本数据类型有nil、string、boolean、number、function类型。

### nil 类型

nil类似于Java中的null ，表示空值。变量第一次赋值为nil。

```
local num
print(num)
num=100
print(num)

```
终端输出：

> nil
> 
> 100

### number (数字)


Number 类型用于表示实数，和 Java里面的 double 类型很类似。可以使用数学函数
math.floor（向下取整） 和 math.ceil（向上取整） 进行取整操作。

```

local order = 3.99
local score = 98.01
print(math.floor(order))
print(math.ceil(score))

```

输出：

> 3
>
> 99

### string 字符串

Lua 中有三种方式表示字符串:
1、使用一对匹配的单引号。例：'hello'。
2、使用一对匹配的双引号。例："abclua
3.字符串还可以用一种长括号（即[[ ]]） 括起来的方式定义

```
ocal str1 = 'hello world'
local str2 = "hello lua"
local str3 = [["add\name",'hello']]
local str4 = [=[string have a [[]].]=]
print(str1) -->output:hello world
print(str2) -->output:hello lua
print(str3) -->output:"add\name",'hello'
print(str4) --

```

### table (表)

Table 类型实现了一种抽象的“关联数组”。“关联数组”是一种具有特殊索引方式的数组，索引通常是字符串（string） 或者 number 类型，但也可以是除 nil 以外的任意类型的值。

```
local corp = {
    web = "www.google.com", --索引为字符串，key = "web",
                              -- value = "www.google.com"
    telephone = "12345678", --索引为字符串
    staff = {"Jack", "Scott", "Gary"}, --索引为字符串，值也是一个表
    100876, --相当于 [1] = 100876，此时索引为数字
            -- key = 1, value = 100876
    100191, --相当于 [2] = 100191，此时索引为数字
    [10] = 360, --直接把数字索引给出
    ["city"] = "Beijing" --索引为字符串
}

print(corp.web) -->output:www.google.com
print(corp["telephone"]) -->output:12345678
print(corp[2]) -->output:100191
print(corp["city"]) -->output:"Beijing"
print(corp.staff[1]) -->output:Jack
print(corp[10]) -->output:36

```

### function(函数)

在 Lua 中，函数 也是一种数据类型，函数可以存储在变量中，可以通过参数传递给其他函
数，还可以作为其他函数的返回值。

```

local function foo()
   print("in the function")
   --dosomething()
   local x = 10
   local y = 20
   return x + y
end
local a = foo --把函数赋给变量
print(a())

--output:
in the function
30
```

## 表达式

> ~=   不等于


逻辑运算符 | 说明
---|---
and | 逻辑与
or  | 逻辑或
not | 逻辑非

- a and b 如果 a 为 nil，则返回 a，否则返回 b;
- a or b 如果 a 为 nil，则返回 b，否则返回 a。

```
local c = nil
local d = 0
local e = 100
print(c and d) -->打印 nil
print(c and e) -->打印 nil
print(d and e) -->打印 100
print(c or d) -->打印 0
print(c or e) -->打印 100
print(not c) -->打印 true
print(not d) --> 打印 false

```

在 Lua 中连接两个字符串，可以使用操作符“..”（两个点）.

```
print("Hello " .. "World") -->打印 Hello World
print(0 .. 1) -->打印 01
```

## 控制语句

单个 if 分支 型

```
x = 10
if x > 0 then
    print("x is a positive number")
end

```

两个分支 if-else 型

```
x = 10
if x > 0 then
    print("x is a positive number")
else
    print("x is a non-positive number")
end
```
多个分支 if-elseif-else 型:

```
score = 90
if score == 100 then
    print("Very good!Your score is 100")
elseif score >= 60 then
    print("Congratulations, you have passed it,your score greater or equal to 60")
    --此处可以添加多个elseif
else
    print("Sorry, you do not pass the exam! ")
end

```


## for 控制结构

Lua 提供了一组传统的、小巧的控制结构，包括用于条件判断的 if 用于迭代的 while、repeat
和 for，本章节主要介绍 for 的使用.

### for 数字型

for 语句有两种形式：数字 for（numeric for） 和范型 for（generic for） 。
数字型 for 的语法如下：

```
for var = begin, finish, step do
--body
end

```

实例1：

```
for i = 1, 5 do
    print(i)
end
-- output:
1 2 3 4 5

```

实例2：

```
for i = 1, 10, 2 do
    print(i)
end
-- output:
1 3 5 7 9

```

### for 泛型

泛型 for 循环通过一个迭代器（iterator） 函数来遍历所有值：

```
-- 打印数组a的所有值

local a = {"a", "b", "c", "d"}
for i, v in ipairs(a) do
    print("index:", i, " value:", v)
end

-- output:
index: 1 value: a
index: 2 value: b
index: 3 value: c
index: 4 value: d

```

lua的入门就到这里，因为lua语法虽少，但细节有很多，不可能花很多时间去研究这个。入个门，遇到问题再去查资料就行了。另外需要说明的是本文大部分内容为复制粘贴于OPenResty 最佳实践，感谢原作者的开源电子书，让我获益匪浅。更多内容请参考：

lua入门教程：http://www.runoob.com/lua/lua-tutorial.html

OPenResty 最佳实践： https://moonbingbing.gitbooks.io/openresty-best-practices/content/index.html