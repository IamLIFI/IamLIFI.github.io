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





### 3. 操作符



|         | Arithmetic | relational | logical |               bitwise                |       compound        |     casting     |
| :------ | :--------: | :--------: | :-----: | :----------------------------------: | :-------------------: | :-------------: |
| boolean |    ----    |    ----    |   ok    | 能使用&, \|, ^<br />不能使用移位和 ~ | 只能使用&=， \|=， ^= |     ------      |
| byte    |            |            |  -----  |                                      |                       | 不能转成boolean |
| char    |            |            |  -----  |                                      |                       | 不能转成boolean |
| short   |            |            | ------  |                                      |                       | 不能转成boolean |
| int     |            |            |  -----  |                                      |                       | 不能转成boolean |
| long    |            |            |  -----  |                                      |                       | 不能转成boolean |
| float   |            |            |  ----   |                -----                 |                       | 不能转成boolean |
| double  |            |            |  ----   |                -----                 |                       | 不能转成boolean |

byte, char, int 在使用算术运算的时候，都会获得一个int 结果，必须显式地转换回原来的类型才能进行再次赋值。

在向下转型的时候，采用的截尾算法。