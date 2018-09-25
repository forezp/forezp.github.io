---
layout: post
title:  "android轮播图"
categories: android
tags:  android
---

* content
{:toc} 

 最近做项目，自己封装了一个图片轮播的组件，主要的思想就采用ViewPager和ScrollGater实现，图片加载用的Imageloader，也可以换其他的，比如Glide.具体封装的组件件源码，这里只说下用法，首先上布局文件。
<!--more-->
 
<img src="http://img.blog.csdn.net/20160708091623612" width = "300"  alt="图片名称" align=center />

```java
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context=".MainActivity" >

    <LinearLayout
        android:id="@+id/root"
        android:layout_width="match_parent"
        android:layout_height="180dp"
        android:orientation="vertical" >

        <com.example.shuffviewdemo.ShufflingView
            android:id="@+id/shuffling_view"
            android:layout_width="match_parent"
            android:layout_height="wrap_content" >
        </com.example.shuffviewdemo.ShufflingView>
    </LinearLayout>

</LinearLayout>

```
 初始化ShufflingView,设置des可见，轮播的指示器在底部。设置点击事件监听。

```java
mShufflingView = (ShufflingView) findViewById(R.id.shuffling_view);
		mShufflingView.setContentDesVisibility(View.VISIBLE);
		mShufflingView.setDotGravity(Gravity.BOTTOM);
		mShufflingView.setOnItemClickListener(mOnShufflingItemClickListener);
		bannerList = new ArrayList<ShufflingItemBean>();
		loadShuffingViewData();

```
设置shufflingView 的beans接收的是一个arraylist对象。
```java

ShufflingItemBean shufflingItemBean=new ShufflingItemBean();
		shufflingItemBean.setImg("http://img3.imgtn.bdimg.com/it/u=4227020988,3565099621&fm=21&gp=0.jpg");
		shufflingItemBean.setUrl("www.baidu.com");//点击跳转到webview
		bannerList.add(shufflingItemBean);
		bannerList.add(shufflingItemBean);
		bannerList.add(shufflingItemBean);
		mShufflingView.updateDatas(bannerList);
```

shufflingView 的监听，具体的跳转和类型，根据需求组件设置。
```java
private  void actionFromType(Context context, ShufflingItemBean item) {
		if (item.getType().equals("-1")) {
			return;
		} else if (item.getType().equals("1")) {// 交易贴

		} else if (item.getType().equals("2")) {// 外部链接

		} else if (item.getType().equals("3")) {// 资讯贴

		} else if (item.getType().equals("4")) {// 应援贴

		} else if (item.getType().equals("5")) {//
			//Intent intent = new Intent(context, WebViewActivity.class);


			//intent.putExtra("url",
			//item.getUrl());


			//intent.putExtra("type", 1);
			//context.startActivity(intent);
		} else {// 其他

		}

	}
```
代码并不难，希望通过我这个例子，一是巩固自己的学习和理解，二是希望更多的人更好的学习，我会再接再厉，写更多的博文。

[源码下载](http://download.csdn.net/detail/forezp/9570165)


