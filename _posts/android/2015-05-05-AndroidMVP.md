---
layout: post
title:  "android MVP"
categories: android
tags:  android
---

* content
{:toc}

##Mvp模式简介

衍生于MVC 模式，降低了耦合性，避免了View(Activity/Fragment)承担了所有的责任，
分担了UI层的职责。<br>

<!--more-->

在MVP模式里通常包含4个要素：

* View:负责绘制UI元素、与用户进行交互(在Android中体现为Activity);
* View interface:需要View实现的接口，View通过View interface与Presenter进行交互，降低耦合，方便进行单元测试;
* Model:负责存储、检索、操纵数据(有时也实现一个Model interface用来降低耦合);
* Presenter:作为View与Model交互的中间纽带，处理与用户交互的负责逻辑。

## 为什么要使用 MVP模式

 在Android开发中，Activity并不是一个标准的MVC模式中的Controller，它的首要职责是加载应用的布局和初始化用户界面，
 并接受并处理来自用户的操作请求，进而作出响应。随着界面及其逻辑的复杂度不断提升，Activity类的 职责不断增加，
 以致变得庞大臃肿。当我们将其中复杂的逻辑处理移至另外的一个类（Presneter）中时，Activity其实就是MVP模式中 View，
 它负责UI元素的初始化，建立UI元素与Presenter的关联（Listener之类），同时自己也会处理一些简单的逻辑（复杂的逻辑交由
 Presenter处理）.<br>

 在实际的开发过程中,往往需求和界面是不确定的，随着开发的不断推进，原来的很多界面基本修改得面目全非，这是许多开发者面临
 的一个非常头疼的问题，MVP在一定程度上了解决了这个问题。

 ##MVP 实战
 （0）UserBean
```

public class User {
    
    private int  id;
    private String name ;
    private String mobile ;
    private String password;
    public int getId() {
        return id;
    }
    public void setId(int id) {
        this.id = id;
    }
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public String getMobile() {
        return mobile;
    }
    public void setMobile(String mobile) {
        this.mobile = mobile;
    }
    public String getPassword() {
        return password;
    }
    public void setPassword(String password) {
        this.password = password;
    }
    @Override
    public String toString() {
        return "User [id=" + id + ", name=" + name + ", mobile=" + mobile + ", password=" + password + "]";
    }
}

```

 （1）IUserModel 用户登录model接口 <br>
 需要实现其接口，一般是读取网络数据，并存在JavaBean 中，并对javabean 有set和get的读写权限。
 <br>
 一般为了需要一个Listerner 来监听网络请求读写数据的情况。


``` 
 public interface IUserModel {
    
    // 用户登录接口
    public void login(String phone, String passwdMd5, final LoginHandler handler);

}

```
 （2）IUserLoginView  用户登录view 接口<br>
	根据需求，VIEw 需要对Model的bean数据进行操作，当登录成功，返回登录人信息情况。
```java
 public interface IUserLoginView extends IUserView {
    public void onUserLoginSuccess(User user);

    public void onUserLoginError(String msg);
}
```

 (3)IUserModel监听类

  Model联网成功后，根据返回情况进行监听，它也起到了传递数据的作用，它将Model的数据传递给
  Presenter ，从而Presenter 来讲数据传递给view
```

public interface LoginHandler extends NetworkHandler {
    public void onLoginSuccess(User user);       // 登录成功

    public void onLoginError(String msg);         // 登录失败
}
```
 (4)UserPresenter

 连接model 层和 view层，处理model和view进行交互。
 
```java
 public class UserPresenter {
    private static final String TAG="UserPresenter";
    
    private IUserModel mUser;      
    private IUserView mView;       


    /**
     * 创建用户模型的主导器
     *
     * @param view 如果不需要进行界面展示则View传入null
     */
    public UserPresenter(@Nullable IUserView view) {
        mUser = UserModel.getInstance();
        mView = view;
    }
    
    
    public void login(String mobile,String password){
        mUser.login(mobile, password, new LoginHandler() {
            
            @Override
            public void onlinkError(String msg) {
                if(mView!=null&&mView instanceof IUserLoginView)
                 ( (IUserLoginView) mView).onUserLoginError(msg);
                
            }
            
            @Override
            public void onLoginSuccess(User u) {
                if (mView != null && mView instanceof IUserLoginView)
                    ((IUserLoginView) mView).onUserLoginSuccess(u);
                
            }
            
            @Override
            public void onLoginError(String msg) {
               if(mView!=null&&mView instanceof IUserLoginView)
                   ((IUserLoginView)mView).onUserLoginError(msg);
                
            }
        });
    }

}
```
（5）View 实现层<br>

```
public class MainActivity extends Activity implements IUserLoginView{
    
    @ViewInject(id=R.id.btn_login,click="login") Button btn_login;
    @ViewInject(id=R.id.et_login_name)EditText et_login_name;
    @ViewInject(id=R.id.et_login_password) EditText et_login_password;
    private Context mContext;
    private UserPresenter userPresenter;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        FinalActivity.initInjectedView(this);
        mContext=this;
        userPresenter=new UserPresenter(this);
    }
    
    public void login(View v){
        String mobile=et_login_name.getText().toString();
        String password=et_login_password.getText().toString();
        String md5Pwd=MD5(password);
        userPresenter.login(mobile, md5Pwd);
        
    }
    
    @Override
    public void onUserLoginSuccess(User user) {
        
        Toast.makeText(mContext, ""+user.toString(), Toast.LENGTH_SHORT).show();
        
    }
    @Override
    public void onUserLoginError(String msg) {
        Toast.makeText(mContext, msg, Toast.LENGTH_SHORT).show();
        
    } 
    
    public final static String MD5(String plain) {  
        try {
            MessageDigest md5 = MessageDigest.getInstance("MD5");
            md5.update(plain.getBytes("UTF-8"));
            byte[] m = md5.digest();
            StringBuilder hex = new StringBuilder(m.length * 2);
            for (byte b : m) {
                if ((b & 0xFF) < 0x10) hex.append("0");
                hex.append(Integer.toHexString(b & 0xFF));
            }
            return hex.toString();
        } catch (Exception e) {
           e.printStackTrace();
            return null;
        }
    }  
}


```


代码并不难，希望通过我这个例子，一是巩固自己的学习和理解，二是希望更多的人更好的学习，我会再接再厉，写
更多的博文。

[源码下载](http://download.csdn.net/detail/forezp/9556104)




