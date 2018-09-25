---
layout: post
title:  "工厂设计模式"
categories: j2ee
tags:  j2ee
---

* content
{:toc}

##工厂设计模式

###一.什么是工厂设计模式

>工厂模式是我们最常用的实例化对象模式了，是用工厂方法代替new操作的一种模式。因为工厂模式就相当于创建实例对象的new，虽然这样做，可能多做一些工作，但会给你系统带来更大的可扩展性和尽量少的修改量。工厂模式最直接的作用就是将创建对象和对象的业务逻辑相分离。

<!--more-->

工厂模式最常见的三种：

* 简单工厂模式
* 工厂方法模式
* 抽象工厂

### 二、简单工厂模式

以下是我看《headefirst设计模式》的工厂模式的笔记。非常有意思，但是篇幅有点长，所以打算总结下，写个简洁版的。

假如你有个pizza店，你可以为客人做很多 pizza。

首先，你需要很多pizza对象，定义一个pizza抽象类：

```
public abstract class Pizza {
    abstract void prepared();
    abstract void bake();
    abstract void cut();
    abstract void box();
}


```
中式 pizza:

```
public class ChineasePizza extends Pizza {
    @Override
    public void prepared() {

    }

    @Override
    public void bake() {

    }

    @Override
    public void cut() {

    }

    @Override
    public void box() {

    }
}
```

美式pizza:

```
public class AmericanPizza extends Pizza {
    @Override
    public void prepared() {

    }

    @Override
    public void bake() {

    }

    @Override
    public void cut() {

    }

    @Override
    public void box() {

    }
}


```
当我需要为客人准备一份pizza，我们可能会需要这样写：

```
public Pizza orderPizza(String type){
        Pizza pizza = null;
        if(type.equals("c")){
            pizza=new ChineasePizza();
        }else if(type.equals("a")){
            pizza=new AmericanPizza();
        }else if(type.equals("e")){
            pizza=new EnglishPizza();
        }else {
            pizza=new ChineasePizza();
        }
        pizza.prepared();
        pizza.bake();
        pizza.cut();
        pizza.box();
        return pizza;
    }

```

当你有很多种pizza的时候，这个else if()特别的长：


```
 if(type.equals("c")){
            pizza=new ChineasePizza();
        }else if(type.equals("a")){
            pizza=new AmericanPizza();
        }else if(type.equals("e")){
            pizza=new EnglishPizza();
        }else if(...){
        ....
        }else {
            pizza=new ChineasePizza();
        }


```

你需要创建一个工厂来管理这些pizza的创建实例:


```

public class SimplePizzaFactory {

    public Pizza createPizza(String type) {

        Pizza pizza = null;
        if(type.equals("c")){
            pizza=new ChineasePizza();
        }else if(type.equals("a")){
            pizza=new AmericanPizza();
        }else if(type.equals("e")){
            pizza=new EnglishPizza();
        }else {
            pizza=new ChineasePizza();
        }

        return pizza;
    }
}

```

有了工厂之后的你代码应该这样写：

```
public class PizzaStore {

    private SimplePizzaFactory simplePizzaFactory;
    public PizzaStore(SimplePizzaFactory simplePizzaFactory){
        this.simplePizzaFactory=simplePizzaFactory;
    }

    public Pizza orderPizza(String type) {
       Pizza pizza= simplePizzaFactory.createPizza(type);
        pizza.prepared();
        pizza.bake();
        pizza.cut();
        pizza.box();
        return pizza;
    }
}

```
就是简单的工厂模式，工厂处理创建对象的细节。工厂模式将创建对象和对象的业务逻辑相分离，降低了代码的耦合性，提高了代码的可读性。

### 三、工厂方法模式

当你需要开很多pizza店的时候，你可能需要创建的pizztore抽象类,这个抽象类只专注于卖pizza的业务逻辑，不专注pizza是怎么来的：

```
public abstract class APizzaStore {

    public Pizza orderPizza(String type) {
        Pizza pizza= createPizza(type);
        pizza.prepared();
        pizza.bake();
        pizza.cut();
        pizza.box();
        return pizza;
    }
    abstract Pizza createPizza(String type);
}


```
它的子类需要专注于pizza 是怎么来的，你需要在北京开一家pizza店：

```
public class BeijingtPizzaStore extends APizzaStore{

    @Override
    Pizza createPizza(String type) {
        Pizza pizza;
        if(type.equals("a")){
            pizza=new AmericanPizza();
        }else {
            pizza=new ChineasePizza();
        }
        return pizza;
    }
}
```

你在北京卖pizza之需要：

```
BeijingtPizzaStore bps=new BeijingtPizzaStore();
bps.orderPizza();

```
 同理,你需要在深圳开pizza店：  
 
```
 public class ShenZhengPizzaStore extends APizzaStore {
    @Override
    Pizza createPizza(String type) {
        return null;
    }
}

```

工厂方法用来处理对象的创建，并将这样的行为封装在子类中。超类的代码和子类的对象创建代码就解耦了。


### 四、抽象工厂模式

你制造pizza的时候，需要很多原料，这时需要一个工厂来制造：

```
public interface IngredientFactory {
    Dauch createDauch();
    Sauce createSauch();
    Cheese createCheese();
}


```

创建一个中国原料工厂：

```
public class ChineaseIngredientFactory implements IngredientFactory {
    @Override
    public Dauch createDauch() {
        return new Dauch();
    }

    @Override
    public Sauce createSauch() {
        return new Sauce();　//这里我简便的写，其实也可以说Sauch的子类。
    }

    @Override
    public Cheese createCheese() {
        return new Cheese();
    }
}


```

同理也可以创建美国原料工厂：

这时我们可以重做pizza:

```

public abstract class Pizza {    
                                 
    String name;                 
    Dauch dauch;                 
    Sauce sauch;            
    Cheese  cheese ;     
                                 
    abstract void prepared();    
    abstract void bake();        
    abstract void cut();         
    abstract void box();         
}                                
                                 
                                                                                  
```


中国式pizza:

```
public class ChineasePizza extends Pizza {
    
    private ChineaseIngredientFactory ingredientFactory;
    
    public ChineasePizza(ChineaseIngredientFactory ingredientFactory){
        this.ingredientFactory=ingredientFactory;
    }
    @Override
    public void prepared() {
        dauch=ingredientFactory.createDauch();
        sauch=ingredientFactory.createSauch();
        cheese=ingredientFactory.createCheese();
        //TODO
        
    }

    @Override
    public void bake() {
        //TODO
    }

    @Override
    public void cut() {
        //TODO
    }

    @Override
    public void box() {
        //TODO
    }
}


```

这时候我们再看，利用抽象工厂模式，可以为pizza创建原料，pizza只关注于它自身的业务逻辑，而不用关注pizza原料从哪里来，这样的pizza和pizza原料的解耦。


> 通过抽象工厂所提供的接口，可以创建产品的家族，利用这个接口写代码，我们的代码将实际工厂解耦，以便在不同的工厂，制造出不同的产品。
> 

到此为止，三种工厂模式已经讲解完毕，[源码下载](http://download.csdn.net/detail/forezp/9757759)。

### 优秀文章推荐：
* [史上最简单的 SpringCloud 教程 | 终章](http://blog.csdn.net/forezp/article/details/70148833)
* [史上最简单的 SpringCloud 教程 | 第一篇: 服务的注册与发现（Eureka）](http://blog.csdn.net/forezp/article/details/69696915)
* [史上最简单的SpringCloud教程 | 第七篇: 高可用的分布式配置中心(Spring Cloud Config)](http://blog.csdn.net/forezp/article/details/70037513)
