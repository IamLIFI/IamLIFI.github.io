---
layout:     post                    # 使用的布局（不需要改）
title:      BaseActivty               # 标题
subtitle:   仿网易云音乐项目包装的类 #副标题
date:       2018-12-1           # 时间
author:     IamLIFI                     # 作者
header-img: img/post-bg-2015.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - Android
    - 仿网易云音乐
---

## 包装类 目的和过程


在仿网易云音乐的项目中，每一个活动都需要执行相似的活动，比如： 初始化控件，为控件设置数据和监听器等等活动，也需要执行数据保存和页面跳转等等的活动。

## 代码和注释
- 每一个活动都需要初始化控件和设置数据与监听器。而且这些还有前后顺序，必须在找到控件之后才能设置数据和设置监听器。
- 继承AppCompatActivity， 然后所有类都要继承BaseActivity.
- 重写setContentView方法， 在setContentView方法中调用init()方法， 这样在子类的活动实例中使用setContentView方法的时候就可以顺便初始化这些控件的数据和监听器。
- 难免要执行页面跳转，所以可以封装startActivity方法，由于子类都继承了BaseActivity, 所以在执行getActivity方法的时候放回的this就是子类的一个引用。



  ```java  
  public class BaseActivity extends AppCompatActivity {

      void initDatas() {}

      void initViews() {}

      void initListeners() {}

      void init() {
          initViews();
          initDatas();
          initListeners();
      }

      @Override
      public void setContentView(int layoutResID) {
          super.setContentView(layoutResID);
          init();
      }
      void startActivity(Class<?> clazz) {
          startActivity(new Intent(getActivity(), clazz));
      }
      void startActivityAfterFinishThis(Class<?> clazz) {
          startActivity(new Intent(getActivity(), clazz));
          finish();
      }


      public BaseActivity getActivity() {
          return this;
      }

      public void hideLoading() {
  //        if (progressDialog != null && progressDialog.isShowing()) {
  //            progressDialog.hide();
  //        }
      }
  }

  ```
