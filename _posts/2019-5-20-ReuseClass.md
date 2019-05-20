---
layout:     post                    # 使用的布局（不需要改）
title:      复用类     # 标题
subtitle:    #副标题
date:       2019-5-20           # 时间
author:     IamLIFI                     # 作者
header-img: img/post-bg-2015.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - Java基础
---

## 组合语法

只需要将对象引用置于新类中即可，基本类型和String类型可以直接使用。

在使用之前，必须要初始化确保正确。

1. 在定义对象的地方。这意味着他们总是能够在构造器被调用之前被初始化。
2. 在类的构造器中。
3. 就在正要使用这些对象之前。
4. 使用实例初始化。

## 继承语法

## 代理

我们将一个成员置于所要构造的类中（像组合），同时暴露了该成员对象的所有方法（像继承）。

```java
public class ShipControls {
    void up();
    void down();
}
public class Ship extends ShipControls {
    //在继承了控制类之后，Ship 就可以控制前后移动了，
    //但是Ship只是想实现移动，他并不是一个 控制类的子类，
    //而且ship 可能不需要可以down的方法，但是这个方法无论是组合还是继承都暴露了出来。
}
/*
* 代理： 可以选择只提供某些子集，比如只提供 up 方法不提供 down 方法
*/
public class Ship {
    private ShipControls controls = new ShipControls();
    //使用复合的方法，生成一个代理类，重写同名函数，然后通过代理类调用原函数。
    public void up() {
        controls.up();
    }
}
```

