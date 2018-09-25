---
layout: post
title:  "android图片圆角实现方式"
categories: android
tags:  android
---

* content
{:toc} 

android 圆角图片的实现形式，包括用第三方、也有系统的。比如makeramen:roundedimageview，系统的cardview ， glide .fresco 。

<!--more-->

![这里写图片描述](http://img.blog.csdn.net/20160819134250608)
```
 compile 'com.android.support:appcompat-v7:24.0.0'
    compile 'com.makeramen:roundedimageview:2.2.1'
    compile 'com.android.support:cardview-v7:24.0.0'
    compile 'com.github.bumptech.glide:glide:3.7.0'
    compile 'com.facebook.fresco:fresco:0.12.0'
```

```
<android.support.v7.widget.CardView
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/id_cardview"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_gravity="center_horizontal"
    app:cardBackgroundColor="@color/bg_light_gray"
    app:cardCornerRadius="3dp"
    app:cardUseCompatPadding="false"
    app:cardPreventCornerOverlap="true"

    >
    <ImageView
        android:id="@+id/iv_subject"
        android:gravity="center"
        android:scaleType="centerCrop"
        android:layout_width="match_parent"
        android:layout_height="200dp" />

    <TextView
        android:paddingLeft="5dp"
        android:paddingBottom="5dp"
        android:background="@drawable/bg_biaoti"
        android:id="@+id/tv_subject"
        android:gravity="center_vertical"
        android:text=""
        android:ellipsize="end"
        android:singleLine="true"
        android:textSize="13sp"
        android:textColor="@color/white"
        android:layout_gravity="bottom"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />

</android.support.v7.widget.CardView>
```

```

 iv_round=(RoundedImageView) findViewById(R.id.iv_round);
 Glide.with(this).load(url).into(iv_round);
```

```
 iv_cardview=(ImageView)findViewById(R.id.iv_cardview);
   Glide.with(this).load(url).into(iv_cardview);
```

```
   iv_fresco=(SimpleDraweeView)findViewById(R.id.iv_fresco);
        Glide.with(this).load(url).into(iv_round);
        Glide.with(this).load(url).into(iv_cardview);
        Uri uri = Uri.parse(url);
        iv_fresco.setImageURI(uri);
```


```
package roundimageview.forezp.com.roundimageview;

import android.content.Context;
import android.content.res.Resources;
import android.graphics.Bitmap;
import android.graphics.BitmapShader;
import android.graphics.Canvas;
import android.graphics.Paint;
import android.graphics.RectF;

import com.bumptech.glide.load.engine.bitmap_recycle.BitmapPool;
import com.bumptech.glide.load.resource.bitmap.BitmapTransformation;

/**
 * Created by Administrator on 2016/8/19 0019.
 */
public class  GlideRoundTransform extends BitmapTransformation {

    private static float radius = 0f;

    public GlideRoundTransform(Context context) {
        this(context, 4);
    }

    public GlideRoundTransform(Context context, int dp) {
        super(context);
        this.radius = Resources.getSystem().getDisplayMetrics().density * dp;
    }

    @Override protected Bitmap transform(BitmapPool pool, Bitmap toTransform, int outWidth, int outHeight) {
        return roundCrop(pool, toTransform);
    }

    private static Bitmap roundCrop(BitmapPool pool, Bitmap source) {
        if (source == null) return null;

        Bitmap result = pool.get(source.getWidth(), source.getHeight(), Bitmap.Config.ARGB_8888);
        if (result == null) {
            result = Bitmap.createBitmap(source.getWidth(), source.getHeight(), Bitmap.Config.ARGB_8888);
        }

        Canvas canvas = new Canvas(result);
        Paint paint = new Paint();
        paint.setShader(new BitmapShader(source, BitmapShader.TileMode.CLAMP, BitmapShader.TileMode.CLAMP));
        paint.setAntiAlias(true);
        RectF rectF = new RectF(0f, 0f, source.getWidth(), source.getHeight());
        canvas.drawRoundRect(rectF, radius, radius, paint);
        return result;
    }

    @Override public String getId() {
        return getClass().getName() + Math.round(radius);
    }
}

```

```
    Glide.with(this).load(url).transform(new GlideRoundTransform(this,6)).into(iv_glide);
```