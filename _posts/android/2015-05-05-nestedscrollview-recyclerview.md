---
layout: post
title:  "NestedScrollView和RecyclerView使用"
categories: android
tags:  android
---

* content
{:toc}


NestedScrollView和RecyclerView使用，并设置间距:

<!--more-->

效果图如下：
![这里写图片描述](http://oaurstf0m.bkt.clouddn.com/DEREEE360.gif)
1.NestedScrollView 和RecyclerView嵌套问题（类似ScrollView 和listView）\
需重写 RecyclerView  的  GridLayoutManager(还有另外2种，随便搜下就有)
```
public class FullyGridLayoutManager extends GridLayoutManager {
    public FullyGridLayoutManager(Context context, int spanCount) {
        super(context, spanCount);
    }

    public FullyGridLayoutManager(Context context, int spanCount, int orientation, boolean reverseLayout) {
        super(context, spanCount, orientation, reverseLayout);
    }

    private int[] mMeasuredDimension = new int[2];

    @Override
    public void onMeasure(RecyclerView.Recycler recycler, RecyclerView.State state, int widthSpec, int heightSpec) {
        final int widthMode = View.MeasureSpec.getMode(widthSpec);
        final int heightMode = View.MeasureSpec.getMode(heightSpec);
        final int widthSize = View.MeasureSpec.getSize(widthSpec);
        final int heightSize = View.MeasureSpec.getSize(heightSpec);

        int width = 0;
        int height = 0;
        int count = getItemCount();
        int span = getSpanCount();
        for (int i = 0; i < count; i++) {
            measureScrapChild(recycler, i,
                    View.MeasureSpec.makeMeasureSpec(i, View.MeasureSpec.UNSPECIFIED),
                    View.MeasureSpec.makeMeasureSpec(i, View.MeasureSpec.UNSPECIFIED),
                    mMeasuredDimension);

            if (getOrientation() == HORIZONTAL) {
                if (i % span == 0) {
                    width = width + mMeasuredDimension[0];
                }
                if (i == 0) {
                    height = mMeasuredDimension[1];
                }
            } else {
                if (i % span == 0) {
                    height = height + mMeasuredDimension[1];
                }
                if (i == 0) {
                    width = mMeasuredDimension[0];
                }
            }
        }

        switch (widthMode) {
            case View.MeasureSpec.EXACTLY:
                width = widthSize;
            case View.MeasureSpec.AT_MOST:
            case View.MeasureSpec.UNSPECIFIED:
        }

        switch (heightMode) {
            case View.MeasureSpec.EXACTLY:
                height = heightSize;
            case View.MeasureSpec.AT_MOST:
            case View.MeasureSpec.UNSPECIFIED:
        }

        setMeasuredDimension(width, height);
    }

    private void measureScrapChild(RecyclerView.Recycler recycler, int position, int widthSpec,
                                   int heightSpec, int[] measuredDimension) {
        if (position < getItemCount()) {
            try {
                View view = recycler.getViewForPosition(0);//fix 动态添加时报IndexOutOfBoundsException
                if (view != null) {
                    RecyclerView.LayoutParams p = (RecyclerView.LayoutParams) view.getLayoutParams();
                    int childWidthSpec = ViewGroup.getChildMeasureSpec(widthSpec,
                            getPaddingLeft() + getPaddingRight(), p.width);
                    int childHeightSpec = ViewGroup.getChildMeasureSpec(heightSpec,
                            getPaddingTop() + getPaddingBottom(), p.height);
                    view.measure(childWidthSpec, childHeightSpec);
                    measuredDimension[0] = view.getMeasuredWidth() + p.leftMargin + p.rightMargin;
                    measuredDimension[1] = view.getMeasuredHeight() + p.bottomMargin + p.topMargin;
                    recycler.recycleView(view);
                }
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }
}
```

重写NestedScrollView,实际上是NestedScrollView禁止滑动

```
public class MyNestedScrollView extends NestedScrollView {
    private int downX;
    private int downY;
    private int mTouchSlop;

    public MyNestedScrollView(Context context) {
        super(context);
        mTouchSlop = ViewConfiguration.get(context).getScaledTouchSlop();
    }

    public MyNestedScrollView(Context context, AttributeSet attrs) {
        super(context, attrs);
        mTouchSlop = ViewConfiguration.get(context).getScaledTouchSlop();
    }

    public MyNestedScrollView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        mTouchSlop = ViewConfiguration.get(context).getScaledTouchSlop();
    }

    @Override
    public boolean onInterceptTouchEvent(MotionEvent e) {
        int action = e.getAction();
        switch (action) {
            case MotionEvent.ACTION_DOWN:
                downX = (int) e.getRawX();
                downY = (int) e.getRawY();
                break;
            case MotionEvent.ACTION_MOVE:
                int moveY = (int) e.getRawY();
                if (Math.abs(moveY - downY) > mTouchSlop) {
                    return true;
                }
        }
        return super.onInterceptTouchEvent(e);
    }
}

```
让recyclerView滑动


 ```java
   recyclerView.setNestedScrollingEnabled(false);
 
 ```
 给recyclerView创建Adapter
 
```
 public class DemoAdapter extends RecyclerView.Adapter<DemoViewHolder> {
    private ArrayList<String> list;
    public Context mContext;

    public DemoAdapter(Context mContext) {
        this.mContext=mContext;
    }
    @Override
    public DemoViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        View mView = LayoutInflater.from(mContext).inflate(R.layout.item_viewholder, parent, false);
        DemoViewHolder mViewHolder = new DemoViewHolder(mView);
        return mViewHolder;
    }

    @Override
    public void onBindViewHolder(DemoViewHolder holder, int position) {
        Glide.with(mContext).load("http://img.nongshanghang.cn/allimg/160906/22435210b_1.jpg").into(  holder.imageView);
    }

    @Override
    public int getItemCount() {
        return 9;
    }
}
 
```
 
 Viewholder 部分
 
```
 public class DemoViewHolder extends RecyclerView.ViewHolder {

    public DemoViewHolder(View itemView) {
        super(itemView);
        imageView=(ImageView)itemView.findViewById(R.id.imageview);
    }
    public ImageView imageView;
}
 
```
 
 设置RecyclerView 的item间距
 
```
 public class SpacesItemDecoration extends RecyclerView.ItemDecoration  {


    private int left;
    private int right;
    private int top;
    private int bottom;
    public SpacesItemDecoration(int space) {
        this.left=space;
        this.right=space;
        this.top=space;
        this.bottom=space;

    }

    public SpacesItemDecoration(int left,int right,int top,int bottom) {
        this.left=left;
        this.right=right;
        this.top=top;
        this.bottom=bottom;

    }

    @Override
    public void getItemOffsets(Rect outRect, View view, RecyclerView parent, RecyclerView.State state) {
        outRect.left=left;
        outRect.right=right;
        outRect.bottom=bottom;
        outRect.top=top;

    }
}
 
```
 
 
 最后设置
 
```
   recyclerView=(RecyclerView)findViewById(R.id.recyclerview);
        recyclerView.setNestedScrollingEnabled(false);
        layoutManager=new FullyGridLayoutManager(this,3,GridLayoutManager.VERTICAL,false);
        adpater=new DemoAdapter(this);
        recyclerView.setAdapter(adpater);
        recyclerView.setLayoutManager(layoutManager);
        SpacesItemDecoration decoration;
        //  if (Integer.parseInt(android.os.Build.VERSION.SDK) >= 19) {
        //     decoration = new SpacesItemDecoration(ScreenUtils.dipToPx(getActivity(), 4), ScreenUtils.dipToPx(getActivity(), 4), ScreenUtils.dipToPx(getActivity(), 0), ScreenUtils.dipToPx(getActivity(), 8));
        //   }else{
        decoration = new SpacesItemDecoration(4, 4, 4,4);
        //  }
        recyclerView.addItemDecoration(decoration);
        adpater.notifyDataSetChanged();
  
```
  代码并不难，希望通过我这个例子，一是巩固自己的学习和理解，二是希望更多的人更好的学习，我会再接再厉，写更多的博文。

[源码下载](http://download.csdn.net/detail/forezp/9624181)

[csdn博客](http://blog.csdn.net/forezp)

