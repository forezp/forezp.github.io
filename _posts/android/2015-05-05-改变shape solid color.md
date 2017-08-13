
---
layout: post
title:  "改变shape里面的值"
categories: android
tags:  android
---

* content
{:toc}

<!--more-->

```
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android">

    <solid android:color="#FF4081"/>
    <corners android:radius="15dp"/>
</shape>
```


```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:paddingBottom="@dimen/activity_vertical_margin"
    android:paddingLeft="@dimen/activity_horizontal_margin"
    android:paddingRight="@dimen/activity_horizontal_margin"
    android:paddingTop="@dimen/activity_vertical_margin"
    tools:context="forezp.com.changesolidcolorfromshape.MainActivity">

    <TextView
        android:id="@+id/tv"
        android:background="@drawable/button"
        android:layout_width="wrap_content"
        android:textSize="14sp"
        android:padding="10dp"
        android:textColor="#ffffff"
        android:layout_height="wrap_content"
        android:text="Hello World!" />
</RelativeLayout>
	
```

```
public class MainActivity extends AppCompatActivity {
    //http://stackoverflow.com/questions/16775891/how-to-change-solid-color-from-the-code
    private TextView tv;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        tv=(TextView)findViewById(R.id.tv);
        GradientDrawable myGrad = (GradientDrawable)tv.getBackground();
        myGrad.setColor(Color.BLACK);
    }
}
```