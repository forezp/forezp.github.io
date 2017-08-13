---
layout: post
title:  "Rxjava +retrofit+mvp"
categories: android
tags:  android
---

* content
{:toc}

最近在看Rxjava，写了一个简单的 demo整合了Rxjava +retrofit+mvp，写完了迫不及待的分享出来了，打算以后的开发都用这个，太强大了，另外OKhTTP用了网络缓存，非常的方便，以后写缓存都不需要写本地数据库了。

<!--more-->

这个项目使用到了拉姆达表达式：
在安卓中使用Java 8 lambda表达式，很遗憾安卓原生不支持，需要用插件。
 在工程中build.GRADLE导入：
 
```
  dependencies {
        classpath 'com.android.tools.build:gradle:2.1.0'
        classpath 'me.tatarka:gradle-retrolambda:3.2.5'
    }

```
 在项目的build.gradle导入
```
  apply plugin: 'me.tatarka.retrolambda'
```
  
 需要使用到Java8在android根下
 
```
  android {
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }
}
```


 
 这样就可以在android下使用拉姆达表达式。
 
 引入RXjAVA、RxAndroid、retrofit  jar包

```
  compile 'io.reactivex:rxandroid:1.2.1'
    compile 'com.squareup.retrofit2:retrofit:2.1.0'
    compile 'com.squareup.retrofit2:converter-gson:2.1.0'
    compile 'com.squareup.retrofit2:adapter-rxjava:2.1.0'
```
 
 关于retrofit的用法，[点击见官网](http://square.github.io/retrofit/)。
 
 关于Rxjava的用法， [翻译的Rxjava中文文档](https://github.com/mcxiaoke/RxDocs)
 
 本文是一个整理好的RxJava+retrofit+Mvp的例子。
 
 本文的的网络请求来源知乎API.
 
 1.首先创建retrofit 接口
 
```
 public interface ZhihuApi {
    @GET("news/latest")
    Observable<NewsTimeLine> getLatestNews();
}

```
 
 2.创建retrofit  service
 
```

  File httpCacheDirectory = new File(MyApp.mContext.getCacheDir(), "responses");
        int cacheSize = 10 * 1024 * 1024; // 10 MiB
        Cache cache = new Cache(httpCacheDirectory, cacheSize);
        OkHttpClient client = new OkHttpClient.Builder()
                .addInterceptor(REWRITE_CACHE_CONTROL_INTERCEPTOR)
                .cache(cache).build();
        Retrofit retrofit_zhihu = new Retrofit.Builder()
                .baseUrl(ZHIHU_BASE_URL)
                .client(client)
                .addConverterFactory(GsonConverterFactory.create())
                .addCallAdapterFactory(RxJavaCallAdapterFactory.create())
                .build();
        ZhihuApiService = retrofit_zhihu.create(ZhihuApi.class);
 

```
配置json解析

```
   .addConverterFactory(GsonConverterFactory.create())
```

配置Rxjava  适配器

```
  .addCallAdapterFactory(RxJavaCallAdapterFactory.create())
```
  其中配置了网络缓存
  
```
   Interceptor REWRITE_CACHE_CONTROL_INTERCEPTOR = chain -> {
        CacheControl.Builder cacheBuilder = new CacheControl.Builder();
        cacheBuilder.maxAge(0, TimeUnit.SECONDS);
        cacheBuilder.maxStale(365, TimeUnit.DAYS);
        CacheControl cacheControl = cacheBuilder.build();
        Request request = chain.request();
        if (!NetUtils.isNetworkAvailable(MyApp.mContext)) {
            request = request.newBuilder()
                    .cacheControl(cacheControl)
                    .build();
        }
        Response originalResponse = chain.proceed(request);
        if (NetUtils.isNetworkAvailable(MyApp.mContext)) {
            int maxAge = 0; // read from cache
            return originalResponse.newBuilder()
                    .removeHeader("Pragma")
                    .header("Cache-Control", "public ,max-age=" + maxAge)
                    .build();
        } else {
            int maxStale = 60 * 60 * 24 * 28; // tolerate 4-weeks stale
            return originalResponse.newBuilder()
                    .removeHeader("Pragma")
                    .header("Cache-Control", "public, only-if-cached, max-stale=" + maxStale)
                    .build();
        }
    };
  
```   

 
 
  在MVP的P层进行网络请求，传统的应该是Model  层，写在P层少一次接口回调。
 
```
   public void getNews(IGetZhihuNewsView iGetZhihuNewsView){
        zhihuApi.getLatestNews( )
                .subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(newsTimeLine -> {
                    disPlayZhihuList(iGetZhihuNewsView,newsTimeLine, context);
                },this::loadError);
    }
   
```
   
其中view接口

```
public interface IGetZhihuNewsView {
    void  getSuccess(NewsTimeLine newsTimeLine);
}

```
  在Activity中
```
  public class MainActivity extends AppCompatActivity implements IGetZhihuNewsView {
    private TextView tv;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        tv= (TextView) findViewById(R.id.tv) ;
        ZhihuPresenter zhihuPresenter=new ZhihuPresenter(this);
        zhihuPresenter.getNews(this);
    }

   @Override
    public void getSuccess(NewsTimeLine newsTimeLine) {
        tv.setText(newsTimeLine.toString());
    }
}
```

  代码比较简单，~~~
  
  [源码下载](https://github.com/forezp/RxJavaRetrofit)
