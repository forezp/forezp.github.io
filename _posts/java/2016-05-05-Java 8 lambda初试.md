---
layout: post
title:  "java8 lambda"
categories: java
tags:  java
---

* content
{:toc}


λ表达式本质上是一个匿名方法。让我们来看下面这个例子：

    public int add(int x, int y) {
        return x + y;
    }


<!--more-->

转成λ表达式后是这个样子：
    
    (int x, int y) -> x + y;

参数类型也可以省略，Java编译器会根据上下文推断出来：

    (x, y) -> x + y; //返回两数之和
 
或者

    (x, y) -> { return x + y; } //显式指明返回值

可见λ表达式有三部分组成：参数列表，箭头（->），以及一个表达式或语句块。

下面这个例子里的λ表达式没有参数，也没有返回值（相当于一个方法接受0个参数，返回void，其实就是Runnable里run方法的一个实现）：

    () -> { System.out.println("Hello Lambda!"); }

如果只有一个参数且可以被Java推断出类型，那么参数列表的括号也可以省略：

    c -> { return c.size(); }
```
public static void main(String[] args) {
		Arrays.asList( "a", "b", "d" ).forEach( e -> {
			System.out.print( e +"\n");

		} );
		System.out.print( "\n--------------------------" );
		Arrays.asList( "a", "b", "d" ).sort( ( e1, e2 ) -> e1.compareTo( e2 ) );


		/**
		 * 可见λ表达式有三部分组成：参数列表，箭头（->），以及一个表达式或语句块。
		 * 下面这个例子里的λ表达式没有参数，也没有返回值（相当于一个方法接受0个参数，返回void，其实就是Runnable里run方法的一个实现）：
		 * () -> { System.out.println("Hello Lambda!"); }
		 */
		Thread t2=new Thread(()->{
			System.out.println("This is from an anonymous method (lambda exp).\n");
		});
		t2.start();
		/**
		 * 迭代LIST
		 */
		List<String> listStr=new ArrayList<>();
		listStr.add("sss");
		listStr.add("1111");
		listStr.forEach(e->{
			if(e.equals("sss")){
				System.out.print(e);
			}
		});

	}
```