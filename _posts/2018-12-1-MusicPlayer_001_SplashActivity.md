---
layout:     post                    # 使用的布局（不需要改）
title:      001_启动页面               # 标题
subtitle:   实现有延迟的跳转 #副标题
date:       2018-12-1           # 时间
author:     IamLIFI                     # 作者
header-img: img/post-bg-2015.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - Android
    - 仿网易云音乐
---

# 项目
>[project](https://github.com/IamLIFI/android/tree/master/RewriteMusic)

## Splash Activity界面思路

![pic1](https://github.com/IamLIFI/IamLIFI.github.io/blob/master/img/1.png?raw=true)

延迟显示的方式：**使用handler发送信息， 根据3种情况跳转**
```java
//创建一个handler
private Handler mHandler = new Handler() {
        public void handleMessage(Message msg) {
            switch (msg.what) {
                case MSG_GUIDE:
                    startActivityAfterFinishThis(GuideActivity.class);
                    break;
                case MSG_HOME:
                    startActivityAfterFinishThis(MainActivity.class);
                    break;
                case MSG_LOGIN:
                    startActivityAfterFinishThis(LoginActivity.class);
                    break;
            }
        }
    };
//然后初始化页面后延迟3秒跳转
@Override
    void initDatas() {
        super.initDatas();
        mHandler.postDelayed(new Runnable() {
            @Override
            public void run() {
                if (isShowGuide()) {
                    mHandler.sendEmptyMessage(MSG_GUIDE);
                } else if(sp.isLogin()){
                    mHandler.sendEmptyMessage(MSG_HOME);
                } else {
                    mHandler.sendEmptyMessage(MSG_LOGIN);
                }
            }
        }, DEFAULT_DELAY_TIME);
    }

```

## 页面全屏的方法
```xml
<!-- 在Mainifest文件中设置主题为NoActionBar，然后再style中隐藏标题栏和状态栏 -->
<activity
     android:name=".activity.SplashActivity"
     android:theme="@style/AppTheme.NoActionBar">
     <intent-filter>
         <action android:name="android.intent.action.MAIN" />

         <category android:name="android.intent.category.LAUNCHER" />
     </intent-filter>
 </activity>

 <style name="AppTheme.NoActionBar">
        <!-- 隐藏标题栏 -->
        <item name="windowNoTitle">true</item>
        <!-- 隐藏状态栏 -->
        <item name="android:windowFullscreen">true</item>
</style>
```

## 在这个页面中使用到的类的包装
[BaseActivity](https://iamlifi.github.io/2018/12/01/BaseActivity/)

[BaseCommonActivity](https://iamlifi.github.io/2018/12/01/BaseCommonActivity-vs-BaseTitleActivty/)

[SharePreference](https://baidu.com)

[PackageUtil](https://baidu.com)
