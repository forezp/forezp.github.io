---
layout: post
title:  "android realm"
categories: android
tags:  android
---

* content
{:toc}

Realm is a mobile database that runs directly inside phones, tablets or wearables. This repository holds the source code for the Java version of Realm, which currently runs only on Android.


<!--more-->

Realm是一个移动端的数据库，它可以在手机、平板。穿戴设备上运行。这个仓库的代码是一个Java版本的代码，目前只用在安卓端。

摘自：https://github.com/realm/realm-java

导入JAR
```
  compile 'io.realm:realm-android:0.87.0'

```
在Application 中配置，不配置也可以，就是默认的哦。

```
public class MyApplication extends Application {

    @Override
    public void onCreate() {
        super.onCreate();
        initRealm();
    }
    private void initRealm(){
        RealmConfiguration configuration = new RealmConfiguration
                .Builder(this)
                .name("test.realm")
                .deleteRealmIfMigrationNeeded()
                .schemaVersion(7).migration(new RealmMigration() {

                    @Override
                    public void migrate(DynamicRealm realm, long oldVersion, long newVersion) {

                    }
                }).build();

        Realm.setDefaultConfiguration(configuration);
    }
}
```
创建实体类，需集成RealmObject
```
public class User  extends RealmObject{
    @PrimaryKey
    private String id;
    private String userName;
    private String mobile;

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public String getUserName() {
        return userName;
    }

    public void setUserName(String userName) {
        this.userName = userName;
    }

    public String getMobile() {
        return mobile;
    }

    public void setMobile(String mobile) {
        this.mobile = mobile;
    }
}
```
 在Activity中初始化
```
Realm myRealm ;
 myRealm= Realm.getInstance(this);
```

添加数据：
```
  //Realm开始处理事物  方式1：
        myRealm.beginTransaction();
        User user = myRealm.createObject(User.class);
        user.setId("445115");
        user.setMobile("44545");
        user.setUserName("hha");
        myRealm.commitTransaction();
        //方式2：
  User user2=new User();
        user2.setId("1123");
        user2.setUserName("sss");
        user2.setMobile("445");
        myRealm.beginTransaction();
        User userCopy2 = myRealm.copyToRealm(user2);
        myRealm.commitTransaction();

```

查找数据

```
 RealmResults<User> listUser = myRealm.where(User.class).findAll();

        StringBuilder stringBuilder=new StringBuilder();

        for(User u:listUser) {
            stringBuilder.append(u.getUserName()+"--------****--------- ");
            Log.d("results1",u.getUserName());
        }
        tv.setText(stringBuilder.toString());

```

代码并不难，希望通过我这个例子，一是巩固自己的学习和理解，二是希望更多的人更好的学习，我会再接再厉，写更多的博文。

[源码下载](https://github.com/forezp/RealmJavaTest)

[csdn博客](http://blog.csdn.net/forezp)


