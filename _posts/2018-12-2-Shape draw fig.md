---
layout:     post                    # 使用的布局（不需要改）
title:      Shape               # 标题
subtitle:   Android画图 #副标题
date:       2018-12-1           # 时间
author:     IamLIFI                     # 作者
header-img: img/post-bg-2015.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - Android
    - 仿网易云音乐
---

> https://blog.csdn.net/lengxuechiwu1314/article/details/72934634
## 通过Shape可以在XML中绘制各种形状，可以定义下面四种类型的形状
![pic1](https://img-blog.csdn.net/20170615143144933?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbGVuZ3h1ZWNoaXd1MTMxNA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

下面是shape的全部属性
```xml
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
    shape = rectangle|矩形，默认的形状，可以画出直角矩形、圆角矩形、弧形等
           oval|椭圆形，用得比较多的是画正圆
           line|线形，可以画实线和虚线
           ring|环形，可以画环形进度条 >

    <!--设置圆角，
    **只适用于rectangle类型**，
    可分别设置四个角不同半径的圆角，当设置的圆角半径很大时，比如200dp，就可变成弧形边了-->
    <corners
        android:radius="integer"//四个角的角度(会被下面四个角度属性覆盖)  
        android:bottomLeftRadius="integer"//四个位置各自的角度
        android:bottomRightRadius="integer"     
        android:topLeftRadius="integer"
        android:topRightRadius="integer"/>

    <!--设置描边，可描成
        实线或虚线
      -->
    <stroke
        android:width="integer"//描边的颜色
        android:color="color"//描边的宽度
        android:dashGap="integer"//设置虚线时的横线之间的距离
        android:dashWidth="integer"/>//设置虚线时的横线长度

    <!--设置形状填充的颜色，只有android:color一个属性-->
    <solid android:color="@color/green"/>

    <!--设置内容与形状边界的内间距，
      **这个图形里面可能是文字或者图片等**
    可分别设置左右上下的距离-->
    <padding
        android:bottom="integer"//下内间距
        android:left="integer"//左内间距
        android:right="integer"//右内间距
        android:top="integer"/>//上内间距

    <!--设置形状的渐变颜色，可以是线性渐变、放射渐变、扫描性渐变-->   
    <gradient
        android:type 渐变的类型
              = linear 线性渐变，默认的渐变类型
                radial 放射渐变，设置该项时，android:gradientRadius也必须设置
                sweep 扫描性渐变
        android:startColor 渐变开始的颜色
        android:endColor 渐变结束的颜色
        android:centerColor 渐变中间的颜色
        android:angle 渐变的角度，线性渐变时才有效，必须是45的倍数，0表示从左到右，90表示从下到上
        android:centerX 渐变中心的相对X坐标，放射渐变时才有效，在0.0到1.0之间，默认为0.5，表示在正中间
        android:centerY 渐变中心的相对X坐标，放射渐变时才有效，在0.0到1.0之间，默认为0.5，表示在正中间
        android:gradientRadius 渐变的半径，只有渐变类型为radial时才使用
        android:useLevel 如果为true，则可在LevelListDrawable中使用
        />

```


### 使用selector
根据不同的状态使用不同的drawable图形
>android:state_selected选中 <br>
android:state_focused获得焦点 <br>
android:state_pressed点击<br>
android:state_enabled设置是否响应事件,指所有事件

```xml
<?xml version="1.0" encoding="utf-8"?>
<selector xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:state_pressed="true" android:drawable="@drawable/shape_button_selected"/>
    <item android:drawable="@drawable/shape_button"/>
</selector>
```

### layer-list

- 其实 layer-list 是用来创建 LayerDrawable 的，LayerDrawable 是 DrawableResource 的一种， 所以，layer-list 创建出来的是 图层列表，也就是一个drawable 图形。

- layer-list 的大致原理类似 RelativeLayout（或者FrameLayout） ，也是一层层的叠加 ，后添加的会覆盖先添加的。在 layer-list 中可以通过 控制后添加图层距离最底部图层的 左上右下的四个边距等属性，得到不同的显示效果。

- 通过画2个高度差距为1dp的矩形，底层的有颜色那么就可以实现一个横线厚度1dp有颜色，且位于视图顶部的一个图形。
