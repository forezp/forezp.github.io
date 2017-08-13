---
layout: post
title:  "android design新控件"
categories: android
tags:  android
---

* content
{:toc}

 最近在研究android   开发的新控件，包括drawer layout ,NavigationView,CoordinatorLayout,AppBarLayout,Toolbar,TabLayout,SwipeRefreshLayout,Recyclerview等。
 
<!--more-->

先上效果图：

<img src="http://img.blog.csdn.net/20160710200140616" width = "300"  alt="图片名称" align=center />     <img src="http://img.blog.csdn.net/20160710200214538" width = "300"  alt="图片名称" align=center />
<img src="http://img.blog.csdn.net/20160710200253055" width = "300"  alt="图片名称" align=center />   <img src="http://img.blog.csdn.net/20160710200353322" width = "300"  alt="图片名称" align=center />

主界面上drawlayou 和NavigationView形成抽屉效果，布局文件如下：
```java
<android.support.v4.widget.DrawerLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/id_drawerlayout"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    >
    <include layout="@layout/home_content"/>
    <android.support.design.widget.NavigationView
        android:id="@+id/id_navigationview"
        app:itemTextColor="@color/selector_nav_menu_textcolor"
        android:layout_gravity="left"
        android:layout_width="match_parent"
        android:layout_height="match_parent">

    </android.support.design.widget.NavigationView>

</android.support.v4.widget.DrawerLayout>
```
java代码：抽屉部分

```
drawerLayout = (DrawerLayout)findViewById(R.id.id_drawerlayout);
navigationView = (NavigationView)findViewById(R.id.id_navigationview);
 ActionBarDrawerToggle mActionBarDrawerToggle =
                new ActionBarDrawerToggle(this, drawerLayout, toolbar, R.string.open, R.string.close);
        mActionBarDrawerToggle.syncState();
        drawerLayout.setDrawerListener(mActionBarDrawerToggle);

        //给NavigationView填充顶部区域，也可在xml中使用app:headerLayout="@layout/header_nav"来设置
        navigationView.inflateHeaderView(R.layout.header_nav);
        View headerView = navigationView.getHeaderView(0);

        CircleImageView circleImageView = (CircleImageView)headerView.findViewById(R.id.id_circleview);
        Glide.with(this).load("http://pic1.nipic.com/2008-10-30/200810309416546_2.jpg").into(circleImageView);

        //给NavigationView填充Menu菜单，也可在xml中使用app:menu="@menu/menu_nav"来设置
        navigationView.inflateMenu(R.menu.menu_nav);

```

可以给navigationview 设置点击事件：

```
mNav.setNavigationItemSelectedListener(new NavigationView.OnNavigationItemSelectedListener() {
            @Override public boolean onNavigationItemSelected(MenuItem menuItem) {

                String msgString = "";

                switch (menuItem.getItemId()) {
                    case R.id.nav_menu_home:
                        msgString = (String) menuItem.getTitle();
                        break;
                    case R.id.nav_menu_categories:
                        msgString = (String) menuItem.getTitle();
                        break;
                    case R.id.nav_menu_feedback:
                        msgString = (String) menuItem.getTitle();
                        break;
                    case R.id.nav_menu_setting:
                        msgString = (String) menuItem.getTitle();
                        break;
                }

                // Menu item点击后选中，并关闭Drawerlayout
                menuItem.setChecked(true);
                drawerLayout.closeDrawers();

                Toast.makeText(HomeActivity.this,msgString,Toast.LENGTH_SHORT).show();
               
                return true;
```
draw layout  和navigation view 组合可以写成非常好的抽屉效果，避免了第三方库，用原生的感觉非常棒。
－－－－－－－－－－抽屉部分结束－－－－－－－－－－－
<br></br>
<br></br>

tab layout 和view pager 实现联动效果：

```
 // 初始化ViewPager的适配器，并设置给它
        mViewPagerAdapter = new MyViewPagerAdapter(getSupportFragmentManager(), mTitles, mFragments);
        viewPager.setAdapter(mViewPagerAdapter);
        // 设置ViewPager最大缓存的页面个数
        viewPager.setOffscreenPageLimit(5);
        // 给ViewPager添加页面动态监听器（为了让Toolbar中的Title可以变化相应的Tab的标题）
        viewPager.addOnPageChangeListener(this);

        tabLayout.setTabMode(MODE_SCROLLABLE);
        // 将TabLayout和ViewPager进行关联，让两者联动起来
        tabLayout.setupWithViewPager(viewPager);
        // 设置Tablayout的Tab显示ViewPager的适配器中的getPageTitle函数获取到的标题
        tabLayout.setTabsFromPagerAdapter(mViewPagerAdapter);

```
RefreshLayout 实现下拉刷新效果:
布局文件：

```
 <android.support.v4.widget.SwipeRefreshLayout xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:app="http://schemas.android.com/apk/res-auto"
        android:id="@+id/id_swiperefreshlayout"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        app:layout_behavior="@string/appbar_scrolling_view_behavior"
        >

        <!--
                使用RecyclerView需要在build.gradle中添加
               compile 'com.android.support:recyclerview-v7:23.3.0'
        -->
        <android.support.v7.widget.RecyclerView
            android:id="@+id/id_recyclerview"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:foregroundGravity="center"
            />


    </android.support.v4.widget.SwipeRefreshLayout>
```
在fragment 实现下拉刷新接口：
```
public class DemoFragment extends Fragment implements SwipeRefreshLayout.OnRefreshListener{}
```
mSwipeRefreshLayout实现下拉是的颜色变化，和设置监听事件。
```
mSwipeRefreshLayout.setColorSchemeResources(R.color.main_blue_light, R.color.main_blue_dark);
 mSwipeRefreshLayout.setOnRefreshListener(this);
```
下拉刷新刷新数据的接口实现的方法：
```
@Override
    public void onRefresh() {

        new Handler().postDelayed(new Runnable() {
            @Override public void run() {
               mSwipeRefreshLayout.setRefreshing(false);//关闭下拉动画

                           
                }
            }
        }, 1000);
    }
```
－－－－－－－－－下拉刷新结束－－－－－－－－－－

RecyclerView可以实现listview （横行和纵向）.gridview（横行和纵向） ,瀑布流的效果。
我讲解一下最简单的效果：listview的效果：
直接上代码：
```
mLayoutManager =new LinearLayoutManager(getActivity(), LinearLayoutManager.VERTICAL, false);
mRecyclerViewAdapter = new DemoRecyclerViewAdapter(getActivity());
mRecyclerViewAdapter.setOnItemClickListener(this);
            mRecyclerView.setAdapter(mRecyclerViewAdapter);
            mRecyclerViewAdapter.setList(list);
            mRecyclerView.setLayoutManager(mLayoutManager);
            mRecyclerViewAdapter.notifyDataSetChanged();                        
```
其中adapter 的写法：
```
public class DemoRecyclerViewAdapter extends RecyclerView.Adapter<DemoRecyclerViewHolder> {
    private Context context;
    private ArrayList<ImageBean> list;
  
    public DemoRecyclerViewAdapter(Context mContext) {
        this.context = mContext;

    }
    @Override
    public DemoRecyclerViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        View mView= LayoutInflater.from(context).inflate(R.layout.item_demo_adapter,parent,false);
        DemoRecyclerViewHolder recyclerViewHolder=new DemoRecyclerViewHolder(mView);
        return recyclerViewHolder;
    }

    @Override
    public void onBindViewHolder(final DemoRecyclerViewHolder holder, final int position) {
        if(mOnItemClickListener!=null){
            holder.itemView.setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View v) {
                    mOnItemClickListener.onItemClick(holder.itemView,position);
                }
            });
            holder.itemView.setOnLongClickListener(new View.OnLongClickListener() {
                @Override
                public boolean onLongClick(View v) {
                    mOnItemClickListener.onItemLongClick(holder.itemView, position);
                    return true;
                }
            });
        }
        holder.textView.setText(list.get(position).getName());
        Glide.with(context).load(list.get(position).getImg()).into(holder.imageView);

    }



    @Override
    public int getItemCount() {
        return list.size();
    }
}
```
adapter的写法根之前BaseAdapter 很类似，需要特别注意的是:
加载布局文件的方法一定是这个，要不然会出现match_parent 失效。
```
View mView= LayoutInflater.from(context).inflate(R.layout.item_demo_adapter,parent,false);
``` 
还有一些其他的控件如cardview 比较简单就不说了，toolbar的用法会在下次给出好的例子。

代码并不难，希望通过我这个例子，一是巩固自己的学习和理解，二是希望更多的人更好的学习，我会再接再厉，写更多的博文。

[源码下载](http://download.csdn.net/detail/forezp/9571175)

### 优秀文章推荐：
* [史上最简单的 SpringCloud 教程 | 终章](http://blog.csdn.net/forezp/article/details/70148833)
* [史上最简单的 SpringCloud 教程 | 第一篇: 服务的注册与发现（Eureka）](http://blog.csdn.net/forezp/article/details/69696915)
* [史上最简单的SpringCloud教程 | 第七篇: 高可用的分布式配置中心(Spring Cloud Config)](http://blog.csdn.net/forezp/article/details/70037513)

