---
layout: post
title:  "viewpager"
categories: android
tags:  android
---

* content
{:toc}


利用简单的Textview 和Viewpager实现滑动、点击换页的效果，效果图如下：

<!--more-->

<img src="http://img.blog.csdn.net/20160625113922931" width = "200"  alt="行走的那些事" align=center />　　　<img src="http://img.blog.csdn.net/20160625113940119" width = "200"  alt="行走的那些事" align=center />

先上布局文件代码：

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@color/white"
    android:orientation="vertical" >

   <LinearLayout
       android:background="@color/red_base"
       android:orientation="horizontal"
       android:layout_width="match_parent"
       android:layout_height="50dp">

   </LinearLayout>
    <!-- 选项卡 -->

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="40dp"
        android:background="@color/white"
        android:orientation="horizontal"
        android:weightSum="5" >

        <FrameLayout
            android:id="@+id/rim_tab1_fl"
            android:layout_width="0dp"
            android:layout_height="match_parent"
            android:layout_weight="1"
            android:clickable="true"
            android:gravity="center" >

            <TextView
                android:id="@+id/rim_tab1_tv"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:layout_gravity="center"
                android:text="全部"
                android:textColor="@color/red_base"
                android:textSize="16sp" />
        </FrameLayout>

        <View
            android:layout_width="0.5dp"
            android:layout_height="match_parent"
            android:layout_marginBottom="10dp"
            android:layout_marginTop="10dp"
            android:background="@color/divider_gray_line" />

        <FrameLayout
            android:id="@+id/rim_tab2_fl"
            android:layout_width="0dp"
            android:layout_height="match_parent"
            android:layout_weight="1"
            android:clickable="true"
            android:gravity="center" >

            <TextView
                android:id="@+id/rim_tab2_tv"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:layout_gravity="center"
                android:text="周边"
                android:textColor="@color/text_gray_4"
                android:textSize="16sp" />
        </FrameLayout>

        <View
            android:layout_width="0.5dp"
            android:layout_height="match_parent"
            android:layout_marginBottom="10dp"
            android:layout_marginTop="10dp"
            android:background="@color/divider_gray_line" />

        <FrameLayout
            android:id="@+id/rim_tab3_fl"
            android:layout_width="0dp"
            android:layout_height="match_parent"
            android:layout_weight="1"
            android:clickable="true"
            android:gravity="center" >

            <TextView
                android:id="@+id/rim_tab3_tv"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:layout_gravity="center"
                android:text="应援"
                android:textColor="@color/text_gray_4"
                android:textSize="16sp" />
        </FrameLayout>

        <View
            android:layout_width="0.5dp"
            android:layout_height="match_parent"
            android:layout_marginBottom="10dp"
            android:layout_marginTop="10dp"
            android:background="@color/divider_gray_line" />

        <FrameLayout
            android:id="@+id/rim_tab4_fl"
            android:layout_width="0dp"
            android:layout_height="match_parent"
            android:layout_weight="1"
            android:clickable="true"
            android:gravity="center" >

            <TextView
                android:id="@+id/rim_tab4_tv"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:layout_gravity="center"
                android:text="话题"
                android:textColor="@color/text_gray_4"
                android:textSize="16sp" />
        </FrameLayout>

        <View
            android:layout_width="0.5dp"
            android:layout_height="match_parent"
            android:layout_marginBottom="10dp"
            android:layout_marginTop="10dp"
            android:background="@color/divider_gray_line" />

        <FrameLayout
            android:id="@+id/rim_tab5_fl"
            android:layout_width="0dp"
            android:layout_height="match_parent"
            android:layout_weight="1"
            android:clickable="true"
            android:gravity="center" >

            <TextView
                android:id="@+id/rim_tab5_tv"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:layout_gravity="center"
                android:text="明星"
                android:textColor="@color/text_gray_4"
                android:textSize="16sp" />
        </FrameLayout>
    </LinearLayout>

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="3dp"
        android:background="@color/white" >

        <ImageView
            android:id="@+id/rim_cursor"
            android:layout_width="80dp"
            android:layout_height="3dp"
            android:layout_marginTop="0dip"
            android:background="@color/title_bar_blue" />
    </LinearLayout>

    <View
        android:layout_width="match_parent"
        android:layout_height="0.1dp"
        android:background="@color/btn_bg_gray" />
    <!-- 选项卡内容显示区域 -->

    <View
        android:layout_width="match_parent"
        android:layout_height="10dp"
        android:background="@color/bg_light_gray" />

    <android.support.v4.view.ViewPager
        android:id="@+id/rim_third_vp"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />

</LinearLayout>
```
上面红色指示器的view的初始化
```
private int screenWidth;//指示器
private ImageView cursorImg;
private LinearLayout.LayoutParams lp;


 private void initViews(){

        WindowManager wm = (WindowManager)
                getSystemService(Context.WINDOW_SERVICE);

        int width = wm.getDefaultDisplay().getWidth();
        screenWidth=width/5;
        cursorImg = (ImageView) findViewById(R.id.rim_cursor);
        lp = (LinearLayout.LayoutParams) cursorImg.getLayoutParams();
        lp.width = screenWidth;
        cursorImg.setLayoutParams(lp);
        leftMargin = lp.leftMargin;
  }
```

初始化indicater

```
private void initViewPager() {
        viewPager = (ViewPager) findViewById(R.id.rim_third_vp);
        fragmentsList = new ArrayList<Fragment>();
        fragment1 = new RimFragment();     
        fragmentsList.add(fragment1);
        fragmentsList.add(fragment2);
        fragmentsList.add(fragment3);
        fragmentsList.add(fragment4);
        fragmentsList.add(fragment5);

        viewPager.setAdapter(new FragmentAdapter(getSupportFragmentManager(),
                fragmentsList));
        viewPager.setCurrentItem(0);
        viewPager.setOnPageChangeListener(this);
    }
```
设置上面选项卡的点击事件
```
 @Override
    public void onClick(View v) {
        switch (v.getId()){
            case R.id.rim_tab1_fl:
                viewPager.setCurrentItem(0);
                num_tab1_tv.setTextColor(getResources().getColor(R.color.red_base));
                num_tab2_tv.setTextColor(getResources().getColor(R.color.text_gray_4));
                num_tab3_tv.setTextColor(getResources().getColor(R.color.text_gray_4));
                num_tab4_tv.setTextColor(getResources().getColor(R.color.text_gray_4));
                num_tab5_tv.setTextColor(getResources().getColor(R.color.text_gray_4));
                fragment1.setMsgName("1","周边");//周边的官方和会员的接口参数,官方
                break;
```

设置viewpager 滑动事件
```
 @Override
    public void onPageScrolled(int position, float positionOffset, int positionOffsetPixels) {

        offset = (screenWidth - cursorImg.getLayoutParams().width) / 5;
        
        hidePoint(position, positionOffsetPixels);//设置红色指示器的位置
        cursorImg.setLayoutParams(lp);
        currentIndex = position;
    }

    @Override
    public void onPageSelected(int position) {

        switch (position){//设置点击事件
            case  0:
                fragment1.setMsgName("1","周边");
                num_tab1_tv.setTextColor(getResources().getColor(R.color.red_base));
                num_tab2_tv.setTextColor(getResources().getColor(R.color.text_gray_4));
                num_tab3_tv.setTextColor(getResources().getColor(R.color.text_gray_4));
                num_tab4_tv.setTextColor(getResources().getColor(R.color.text_gray_4));
                num_tab5_tv.setTextColor(getResources().getColor(R.color.text_gray_4));
                break;
          
           
        }

        if (position==0)
        {


        }else {

        }

    }

    @Override
    public void onPageScrollStateChanged(int state) {

    }

	//设置指示器的位置
    private void hidePoint(int position, int positionOffsetPixels) {
        if (position == 0) {// 0<->1
            lp.leftMargin = (int) (positionOffsetPixels / 5) + offset;

        } else if (position == 1) {// 1<->2
            lp.leftMargin = (int) (positionOffsetPixels / 5) + screenWidth
                    + offset;

        }else  if(position==2){
            lp.leftMargin = (int) (positionOffsetPixels / 5) + screenWidth*2
                    + offset;
        }
        else  if(position==3){
            lp.leftMargin = (int) (positionOffsetPixels / 5) + screenWidth*3
                    + offset;
        }
        else  if(position==4){
            lp.leftMargin = (int) (positionOffsetPixels / 5) + screenWidth*4
                    + offset;
        }
    }

```

代码并不难，希望通过我这个例子，一是巩固自己的学习和理解，二是希望更多的人更好的学习，我会再接再厉，写更多的博文。

[源码下载](http://download.csdn.net/detail/forezp/9559206)







