---
layout:     post                    # 使用的布局（不需要改）
title:      SharedPreferencesUtil               # 标题
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
  仿网易云音乐项目包装SharePreference的一个工具类， 方便使用和节约代码。

## SharePreference 的原理
> 参考 https://www.jianshu.com/p/4984f66f9a4b

- SharedPreferences是Android提供的**数据持久化**的一种手段，适合**单进程、小批量**的数据存储与访问。为什么这么说呢？因为SharedPreferences的实现是基于**单个xml文件**实现的，并且，所有持久化数据都是**一次性加载到内存**，如果数据过大，是不合适采用SharedPreferences存放的。而适用的场景是**单进程的**原因同样如此，由于Android原生的文件访问并**不支持多进程互斥**，所以SharePreferences也不支持，如果多个进程更新同一个xml文件，就可能存在同不互斥问题.

#### SharedPreferences的实现原理之:持久化数据的加载
```java
mSharedPreferences = context.getSharedPreferences("test", Context.MODE_PRIVATE);
    SharedPreferences.Editor editor = mSharedPreferences.edit();
    editor.putString(key, value);
    editor.apply();
```

context.getSharedPreferences其实就是简单的调用ContextImpl的getSharedPreferences，具体实现如下
```java
@Override
public SharedPreferences getSharedPreferences(String name, int mode) {
    SharedPreferencesImpl sp;
    synchronized (ContextImpl.class) {
        if (sSharedPrefs == null) {
            sSharedPrefs = new ArrayMap<String, ArrayMap<String, SharedPreferencesImpl>>();
        }

        final String packageName = getPackageName();
        ArrayMap<String, SharedPreferencesImpl> packagePrefs = sSharedPrefs.get(packageName);
        if (packagePrefs == null) {
            packagePrefs = new ArrayMap<String, SharedPreferencesImpl>();
            sSharedPrefs.put(packageName, packagePrefs);
        }
        sp = packagePrefs.get(name);
        if (sp == null) {
        <!--读取文件-->
            File prefsFile = getSharedPrefsFile(name);
            sp = new SharedPreferencesImpl(prefsFile, mode);
            <!--缓存sp对象-->
            packagePrefs.put(name, sp);
            return sp;
        }
    }
    <!--跨进程同步问题-->
    if ((mode & Context.MODE_MULTI_PROCESS) != 0 ||
        getApplicationInfo().targetSdkVersion < android.os.Build.VERSION_CODES.HONEYCOMB) {
        sp.startReloadIfChangedUnexpectedly();
    }
    return sp;
}
```
先去内存中查询与xml对应的SharePreferences是否已经被创建加载，如果没有那么该创建就创建，该加载就加载，在加载之后，要将所有的key-value保存到内幕才能中去，当然，如果首次访问，可能连xml文件都不存在，那么还需要创建xml文件，与SharePreferences对应的xml文件位置一般都在/data/data/包名/shared_prefs目录下，后缀一定是.xml
![pic2](https://upload-images.jianshu.io/upload_images/1460468-c30485e5d121f874.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/833/format/webp)

这里面数据的加载的地方需要看下，比如，SharePreferences数据的加载是同步还是异步？数据加载是new SharedPreferencesImpl对象时候开始的，
```java
SharedPreferencesImpl(File file, int mode) {
    mFile = file;
    mBackupFile = makeBackupFile(file);
    mMode = mode;
    mLoaded = false;
    mMap = null;
    startLoadFromDisk();
}
```

startLoadFromDisk很简单，就是读取xml配置，如果其他线程想要在读取之前就是用的话，就会被阻塞，一直wait等待，直到数据读取完成

## SharedPreferences的实现原理之:持久化数据的更新
通常更新SharedPreferences的时候是首先获取一个SharedPreferences.Editor，利用它缓存一批操作，之后当做事务提交，有点类似于数据库的批量更新：
```java
  SharedPreferences.Editor editor = mSharedPreferences.edit();
  editor.putString(key1, value1);
  editor.putString(key2, value2);
  editor.putString(key3, value3);
  editor.apply();//或者commit
```
Editor是一个接口，这里的实现是一个EditorImpl对象，它首先批量预处理更新操作，之后再提交更新，在提交事务的时候有两种方式，一种是apply，另一种commit，两者的区别在于：何时将数据持久化到xml文件，**前者是异步的，后者是同步的**。Google推荐使用前一种，因为，就单进程而言，只要保证内存缓存正确就能保证运行时数据的正确性，而**持久化不必太及时**，这种手段在Android中使用还是很常见的，比如权限的更新也是这样.

## SharedPreferencesUtil代码
```java
public class SharedPreferencesUtil {

    public static final String TAG = "SharedPreferencesUtil";
    private static final String USER_TOKEN = "USER_TOKEN";
    private static final String USER_ID = "USER_ID";
    private static final String USER_IM_TOKEN = "USER_IM_TOKEN";


    private static SharedPreferences mPreferences;
    private static SharedPreferences.Editor mEditor;
    private static SharedPreferencesUtil mSharedPreferencesUtil;
    private final Context context;

    //由于需要使用context getSharedPreferences方法，
    //所以在创建的时候传入一个context
    public SharedPreferencesUtil(Context context) {
        this.context = context.getApplicationContext();
        mPreferences = this.context.getSharedPreferences(TAG, Context.MODE_PRIVATE);
        mEditor = mPreferences.edit();
    }
    //使用getInstance 方法来获取一个工具
    public static SharedPreferencesUtil getInstance(Context context) {
        if (mSharedPreferencesUtil == null) {
            mSharedPreferencesUtil = new SharedPreferencesUtil(context);
        }
        return mSharedPreferencesUtil;

    }
    public static SharedPreferencesUtil getCurrentInstance() {
        return  mSharedPreferencesUtil;
    }

    public void put(String key, String value) {
        mEditor.putString(key,value);
        mEditor.commit();
    }

    public void putBoolean(String key,boolean value) {
        mEditor.putBoolean(key,value);
        mEditor.commit();
    }

    public String get(String key) {
        return mPreferences.getString(key,"");
    }

    public boolean getBoolean(String key,boolean defaultValue) {
        return mPreferences.getBoolean(key,defaultValue);
    }

    public void removeSP(String key) {
        mEditor.remove(key);
        mEditor.commit();
    }
    //判断用户是否已经登录了
    //使用"USER_TOKEN" 来记录这个对象
    public boolean isLogin() {
        return !TextUtils.isEmpty(get(USER_TOKEN));
    }

    public  void setToken(String token) {
        put(USER_TOKEN,token);
    }

    public String getToken() {
        return get(USER_TOKEN);
    }

    public String getUserId() {
        return get(USER_ID);
    }

    public void setUserId(String userId) {
        put(USER_ID,userId);
    }

    public String getIMToken() {
        return get(USER_IM_TOKEN);
    }

    public  void setIMToken(String token) {
        put(USER_IM_TOKEN,token);
    }
}

```
