---
layout:     post                    # 使用的布局（不需要改）
title:      BaseCommonActivty vs BaseTitleActivty               # 标题
subtitle:   仿网易云音乐项目包装的类 #副标题
date:       2018-12-1           # 时间
author:     IamLIFI                     # 作者
header-img: img/post-bg-2015.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - Android
    - 仿网易云音乐
---

## 包装类的说明

- 这2个类继承[BaseActivty](https://iamlifi.github.io/2018/12/01/BaseActivity/)
- 有些活动需要标题栏(BaseTitleActivty)，有些不需要标题栏(BaseCommonActivty)
- 在BaseCommonActivty中初始化一些数据处理类
- 在BaseTitleActivty中初始化一些开启标题栏的操作

## 代码
```java
//BaseCommonActivty
public class BaseCommonActivity extends BaseActivity {

    SharedPreferencesUtil sp;

    @Override
    void initDatas() {
        super.initDatas();
        sp = SharedPreferencesUtil.getInstance(getApplicationContext());
    }
}

//BaseTitleActivty
public class BaseTitleActivity extends BaseCommonActivity {

    protected Toolbar toolbar;

    @Override
    void initViews() {
        super.initViews();
        toolbar = findViewById(R.id.toolbar);
        setSupportActionBar(toolbar);
    }
    //是否开启标题栏的返回按钮
    protected void enableBackMenu() {
        getSupportActionBar().setDisplayHomeAsUpEnabled(true);
    }

    @Override
    public void setTitle(CharSequence title) {
        if (!TextUtils.isEmpty(title)) {
            super.setTitle(title);
        }

    }
}

 ```
