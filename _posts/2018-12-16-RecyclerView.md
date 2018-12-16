---
layout:     post                    # 使用的布局（不需要改）
title:      RecyclerView               # 标题
subtitle:    #副标题
date:       2018-12-16           # 时间
author:     IamLIFI                     # 作者
header-img: img/post-bg-2015.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - Android
    - 仿网易云音乐
---


## RecyclerView 的优点
> https://www.jianshu.com/p/29198d6a9049

根据官方的介绍RecylerView是ListView的升级版，既然如此那RecylerView必然有它的优点，现就RecylerView相对于ListView的优点罗列如下：
- RecylerView封装了viewholder的回收复用，也就是说RecylerView标准化了ViewHolder，编写`Adapter`面向的是`ViewHolder`而不再是`View`了，复用的   逻辑被封装了，写起来更加简单。
- 提供了一种插拔式的体验，高度的解耦，异常的灵活，针对一个Item的显示RecylerView专门抽取出了相应的类，来控制Item的显示，使其的扩展性非常强。例如：你想**控制横向或者纵向滑动列表效果**可以通过LinearLayoutManager这个类来进行控制(与GridView效果对应的是GridLayoutManager,与瀑布流对应的还有StaggeredGridLayoutManager等)，也就是说RecylerView不再拘泥于ListView的线性展示方式，它也可以实现GridView的效果等多种效果。你想控制**Item的分隔线**，可以通过继承RecylerView的ItemDecoration这个类，然后针对自己的业务需求去抒写代码。
- 可以控制Item**增删的动画**，可以通过ItemAnimator这个类进行控制，当然针对增删的动画，RecylerView有其自己默认的实现。


## RecyclerView 的分割线
> https://www.jianshu.com/p/64a0021394bb

### 实现方式
- 为item布局设置一个背景色，再为item根标签设置一个margin或者padding，这样就形成了分隔线的效果.
- 在Item布局文件最后加一条横线，为它设置一个背景色，形成分隔线的效果.
然后就是正式的写法了：
- RecyclerView中可以通过`addItemDecoration`()方法添加分割线， 该方法的参数为`RecyclerView.ItemDecoration`，该类为抽象类，官方目前只提供了**一个**实现类`DividerItemDecoration`
```java
//在设置完adapter的时候可以设置这个
//初始化分隔线、添加分隔线
mDivider = new DividerItemDecoration(this,DividerItemDecoration.VERTICAL);
mRecyclerView.addItemDecoration(mDivider);
```
![pic1](https://upload-images.jianshu.io/upload_images/5586232-9575f2461d74b6fc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/720/format/webp)

### 抽象类RecyclerView.ItemDecoration源码
```java
public static abstract class ItemDecoration {

        public void onDraw(Canvas c, RecyclerView parent, State state) {
            onDraw(c, parent);
        }
        @Deprecated
        public void onDraw(Canvas c, RecyclerView parent) {
        }

        public void onDrawOver(Canvas c, RecyclerView parent, State state) {
            onDrawOver(c, parent);
        }
        @Deprecated
        public void onDrawOver(Canvas c, RecyclerView parent) {
        }


        @Deprecated
        public void getItemOffsets(Rect outRect, int itemPosition, RecyclerView parent) {
            outRect.set(0, 0, 0, 0);
        }
        public void getItemOffsets(Rect outRect, View view, RecyclerView parent, State state) {
            getItemOffsets(outRect, ((LayoutParams) view.getLayoutParams()).getViewLayoutPosition(),
                    parent);
        }
    }

```
当我们调用addItemDecoration()方法添加decoration的时候，RecyclerView就会调用该类的onDraw方法去绘制分隔线，也就是说：分隔线是绘制出来的. 下面来了解分隔线是如何绘制出来的.

### 理解分隔线是如何绘制的
系统默认实现类DividerItemDecoration涉及到clipToPadding属性、画布的裁剪一些知识，不太容易理解，也不方便修改，这里介绍网上通用的实现.
首先要理解一个概念：分隔线本质是一个矩形，这个“线”是有长度和宽度的.

**看注释** 读懂代码。 **对比一下DividerItemDecoration的代码其实没有改变什么，只是改了类名，重点在App.Theme主题中改变了`<item name="android:listDivider">@drawable/my_divider</item>`**

```java
/**
* 默认分隔线实现类只支持布局管理器为 LinearLayoutManager
*/
public class CommonItemDecoration extends RecyclerView.ItemDecoration {
    public static final int HORIZONTAL = LinearLayout.HORIZONTAL;
    public static final int VERTICAL = LinearLayout.VERTICAL;

    //使用系统主题中的R.attr.listDivider作为Item间的分割线
    private static final int[] ATTRS = new int[]{ android.R.attr.listDivider};

    private Drawable mDivider;

    private int mOrientation;//布局方向，决定绘制水平分隔线还是竖直分隔线

    private final Rect mBounds = new Rect();


    public CommonItemDecoration (Context context, int orientation) {
        final TypedArray a = context.obtainStyledAttributes(ATTRS);
        mDivider = a.getDrawable(0);
        a.recycle();
        setOrientation(orientation);
    }

    public void setOrientation(int orientation) {
        if (orientation != HORIZONTAL && orientation != VERTICAL) {
            throw new IllegalArgumentException(
                    "Invalid orientation. It should be either HORIZONTAL or VERTICAL");
        }
        mOrientation = orientation;
    }

    /**
     * 一个app中分隔线不可能完全一样，你可以通过这个方法传递一个Drawable 对象来定制分隔线
     */
    public void setDrawable(Drawable drawable) {
        if (drawable == null) {
            throw new IllegalArgumentException("Drawable cannot be null.");
        }
        mDivider = drawable;
    }

    /**
     * 画分隔线
     */
    @Override
    public void onDraw(Canvas c, RecyclerView parent, RecyclerView.State state) {
        if (parent.getLayoutManager() == null) {
            return;
        }
        if (mOrientation == VERTICAL) {
            drawVertical(c, parent);
        } else {
            drawHorizontal(c, parent);
        }
    }

    /**
     * 在LinearLayoutManager方向为Vertical时，画分隔线
     */
    public void drawVertical(Canvas canvas, RecyclerView parent) {
        final int left = parent.getPaddingLeft();//★分隔线的左边 = paddingLeft值
        final int right = parent.getWidth() - parent.getPaddingRight();//★分隔线的右边 = RecyclerView 宽度－paddingRight值
//分隔线不在RecyclerView的padding那一部分绘制

        final int childCount = parent.getChildCount();//★分隔线数量=item数量
        for (int i = 0; i < childCount; i++) {
            final View child = parent.getChildAt(i);//确定是第几个item
            final RecyclerView.LayoutParams params =(RecyclerView.LayoutParams) child.getLayoutParams();
            final int top = child.getBottom() + params.bottomMargin;//★分隔线的上边 = item的底部 + item根标签的bottomMargin值
            final int bottom = top + mDivider.getIntrinsicHeight();//★分隔线的下边 = 分隔线的上边 + 分隔线本身高度
            mDivider.setBounds(left, top, right, bottom);
            mDivider.draw(canvas);
        }
    }


    /**
     * 在LinearLayoutManager方向为Horizontal时，画分隔线
     *      理解了上面drawVertical()方法这个方法也就理解了
     */
    public void drawHorizontal(Canvas canvas, RecyclerView parent) {
        final int top = parent.getPaddingTop();
        final int bottom = parent.getHeight() - parent.getPaddingBottom();

        final int childCount = parent.getChildCount();
        for (int i = 0; i < childCount; i++) {
            final View child = parent.getChildAt(i);
            final RecyclerView.LayoutParams params = (RecyclerView.LayoutParams) child.getLayoutParams();
            final int left = child.getRight() + params.rightMargin;
            final int right = left + mDivider.getIntrinsicHeight();
            mDivider.setBounds(left, top, right, bottom);
            mDivider.draw(canvas);
        }
    }

    /**
     *  获取Item偏移量
     *    此方法是为每个Item四周预留出空间，从而让分隔线的绘制在预留的空间内
     */
   @Override
    public void getItemOffsets(Rect outRect, View view, RecyclerView parent,
            RecyclerView.State state) {
        if (mOrientation == VERTICAL) {//竖直方向的分隔线：item向下偏移一个分隔线的高度
            outRect.set(0, 0, 0, mDivider.getIntrinsicHeight());
        } else {//水平方向的分隔线：item向右偏移一个分隔线的宽度
            outRect.set(0, 0, mDivider.getIntrinsicWidth(), 0);
        }
    }
}

```

### 改变分割线的样式
更改分隔线的样式
DividerItemDecoration画的分割线是读取系统的属性android.R.attr.listDivider，使用系统的listDivider好处就是就是方便我们去随意的分隔线的样式
![pic11](https://upload-images.jianshu.io/upload_images/5586232-f223339093f527a5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/720/format/webp)

- 找到res/values/styles.xml,在其中声明android:listDivider属性，然后使用我们自己的样式
```xml
 <item name="android:listDivider">@drawable/my_divider</item>
```
- 在res/drawable目录下声明我们自己的样式my_divider.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="rectangle" >
    <gradient
        android:centerColor="#ff00ff"
        android:endColor="#00ff00"
        android:startColor="#0000ff"
        android:type="linear" />
    <size android:height="4dp"/>
</shape>
```
- 当然，这样一修改就改变整个app中的分隔线效果了，如果只是想改变某个列表中的分隔线效果，完全可以通过分隔线的setDrawable方法来修改.
也就是说，虽然你在style.xml中设置了颜色A，但是你在java文件中使用这个方法就可以改成颜色B
```java
mDivider = new DividerItemDecoration(this, DividerItemDecoration.VERTICAL);
mDivider.setDrawable(getResources().getDrawable(R.drawable.my_divider));
mRecyclerView.addItemDecoration(mDivider);
```

## RecyclerView.Adapter 的使用

RecyclerView.Adapter，需要实现3个方法：
- onCreateViewHolder()
这个方法主要生成为每个`Item inflater`出一个`View`，但是该方法返回的是一个`ViewHolder`。<u>该方法把`View`直接封装在`ViewHolde`r中，然后我们面向的是ViewHolder这个实例</u>，当然这个ViewHolder需要我们自己去编写。直接省去了当初的convertView.setTag(holder)和convertView.getTag()这些繁琐的步骤。
②onBindViewHolder()
这个方法主要用于适配渲染数据到View中。方法提供给你了一个viewHolder，而不是原来的convertView。
③getItemCount()
这个方法就类似于BaseAdapter的getCount方法了，即总共有多少个条目。
实例：接着来几个小的实例帮助大家更深入的了解RecyclerView的用法，首先来实现一个最简单的列表，效果如



## RecyclerView 的布局
