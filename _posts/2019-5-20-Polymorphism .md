---
layout:     post                    # 使用的布局（不需要改）
title:      多 态     # 标题
subtitle:    #副标题
date:       2019-5-20           # 时间
author:     IamLIFI                     # 作者
header-img: img/post-bg-2015.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - Java基础
---



多态， 也称作动态绑定，后期绑定和运行时绑定。

## 向上转型

某个对象的引用视为对其基类型的引用的做法---叫做*向上转型*

- 前期绑定， 在程序执行前进行绑定。
  在Java中，只有 **域， 静态， final（含private）**是前期绑定，其他的都是后期绑定。

  ```java
  
  ```

- 后期绑定（动态绑定， 运行时绑定），指在运行时根据对象的类型进行绑定。

  也就是说，编译器一直不知道对象的类型，但是方法调用机制能够找到正确的方法体，并且加以调用。
  换一种说法，程序发送信息给某一个对象，让该对象区判断应该怎么做。


## 构造器内部的多态

- 构造器的调用顺序

  1. 程序入口   `public static void main`

  2. 调用`new SubClass()`

  3. 基类构造器 和 导出类构造器 的初始化顺序

     |        | 静态 | 属性 | 构造器 |
     | ------ | ---- | ---- | ------ |
     | 基类   | 1    | 3    | 4      |
     | 构造器 | 2    | 5    | 6      |

- 继承和清理
  初始化和销毁的顺序相反

- 构造器内部的多态方法的行为
  如果在一个构造器的内部调用正在构造的对象的某个动态绑定方法，那情况比较有趣。

  ```java
  class Glyph {
      void draw() { print("Glyph draw");}
      Glyph() {
          print("Glyph before draw");
          draw();
          print("Glyph after draw");
      }
  }
  
  class RoundGlyph extends Glyph {
      private int radius = 1;
      RoundGlyph(int r){
          radius = r;
          print("RoundGlyph.RoundGlyph(), radius = " + radius);
      }
      void draw() {
          print("RoundGlyph.draw(), radius = " + radius);
      }
  }
  ...
      public static void main(String[]args) {
      	new RoundGlyph(5);
  	}
  
  //Output
  Glyph before draw
  RoudnGlyph.draw(), radius = 0    // -------注意这里为0
  Glyph after draw
  RoundGlyph.RoundGlyph(), radius = 5
  ```

  在发生任何事物之前， 将分配给对象的存储空间初始化为二进制0；

  调用被覆盖的**draw()**方法，由于前一步的关系，radius被初始化为0；

  按照声明的顺序调用成员的初始化方法；

  调用导出类的构造器主体。



## 向下转型

```java
class Useful {
    void A() {}
}
class MoreUseful extends Useful{
    void A() {}
    void B() {}
}

Useful[] x = {
    new Useful();
    new MoreUseful();
}
((MoreUseful)x[0]).B(); //  失败抛出异常
((MoreUseful)x[1]).B();  //  向下转型成功。
```

