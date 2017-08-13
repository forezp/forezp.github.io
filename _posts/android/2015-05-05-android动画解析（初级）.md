---
layout: post
title:  "android 动画"
categories: android
tags:  android
---

* content
{:toc}


ObjectAnimator继承自ValueAnimator的，底层的动画实现机制也是基于ValueAnimator来完成的，因此ValueAnimator仍然是整个属性动画当中最核心的一个类。那么既然是继承关系，说明ValueAnimator中可以使用的方法在ObjectAnimator中也是可以正常使用的，它们的用法也非常类似.

<!--more-->

1.旋转控件：
```
 ObjectAnimator animator = ObjectAnimator.ofFloat(textView, "rotation", 0f, 360f);
                animator.setDuration(5000);
                animator.start();
```

2.平移控件
```
float curTranslationX = textView.getTranslationX();
                ObjectAnimator animator = ObjectAnimator.ofFloat(textView, "translationX", curTranslationX, -500f, curTranslationX);
                animator.setDuration(5000);
                animator.start();
```
3.放大缩小

```
 ObjectAnimator animator = ObjectAnimator.ofFloat(textView, "scaleY", 1f, 3f, 1f);
                animator.setDuration(3000);
                animator.start();
```

4.透明  控件
```
  ObjectAnimator animator = ObjectAnimator.ofFloat(textView, "alpha", 1f, 0f,1f);
                animator.setDuration(3000);
                animator.start();
```
用法确实比较简单。
5.组合动画
实现组合动画功能主要需要借助AnimatorSet这个类，这个类提供了一个play()方法，如果我们向这个方法中传入一个Animator对象(ValueAnimator或ObjectAnimator)将会返回一个AnimatorSet.Builder的实例，AnimatorSet.Builder中包括以下四个方法：
after(Animator anim)   将现有动画插入到传入的动画之后执行
after(long delay)   将现有动画延迟指定毫秒后执行
before(Animator anim)   将现有动画插入到传入的动画之前执行
with(Animator anim)   将现有动画和传入的动画同时执行

```
 ObjectAnimator moveIn = ObjectAnimator.ofFloat(textView, "translationX", -500f, 0f);
                ObjectAnimator rotate = ObjectAnimator.ofFloat(textView, "rotation", 0f, 360f);
                ObjectAnimator fadeInOut = ObjectAnimator.ofFloat(textView, "alpha", 1f, 0f, 1f);
                AnimatorSet animSet = new AnimatorSet();
                AnimatorSet.Builder builder=animSet.play(rotate);
                builder.with(fadeInOut).after(moveIn);
                animSet.setDuration(5000);
                animSet.start();
```

布局实现

```
<?xml version="1.0" encoding="utf-8"?>

<set xmlns:android="http://schemas.android.com/apk/res/android"
    android:ordering="sequentially" >

    <objectAnimator
        android:duration="2000"
        android:propertyName="translationX"
        android:valueFrom="-500"
        android:valueTo="0"
        android:valueType="floatType" >
    </objectAnimator>

    <set android:ordering="together" >
        <objectAnimator
            android:duration="3000"
            android:propertyName="rotation"
            android:valueFrom="0"
            android:valueTo="360"
            android:valueType="floatType" >
        </objectAnimator>

        <set android:ordering="sequentially" >
            <objectAnimator
                android:duration="1500"
                android:propertyName="alpha"
                android:valueFrom="1"
                android:valueTo="0"
                android:valueType="floatType" >
            </objectAnimator>
            <objectAnimator
                android:duration="1500"
                android:propertyName="alpha"
                android:valueFrom="0"
                android:valueTo="1"
                android:valueType="floatType" >
            </objectAnimator>
        </set>
    </set>

</set>
```
代码：
```
Animator animator = AnimatorInflater.loadAnimator(MainActivity.this, R.animator.setanim);
                animator.setTarget(textView);
                animator.start();
```
调用AnimatorInflater的loadAnimator来将XML动画文件加载进来，然后再调用setTarget()方法将这个动画设置到某一个对象上面，最后再调用start()方法启动动画就可以了，就是这么简单。



[源码下载](http://download.csdn.net/detail/forezp/9587802)

[csdn博客](http://blog.csdn.net/forezp)


 本文参考了 郭神的博客：http://blog.csdn.net/guolin_blog/article/details/43536355
