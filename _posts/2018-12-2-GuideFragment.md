---
layout:     post                    # 使用的布局（不需要改）
title:      GuideFragment               # 标题
subtitle:   用于仿网易云音乐引导页面的ViewPager #副标题
date:       2018-12-2           # 时间
author:     IamLIFI                     # 作者
header-img: img/post-bg-2015.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - Android
    - 仿网易云音乐
---

> https://www.jianshu.com/p/11c8ced79193

## 基本概念
- Fragment是依赖于Activity的，不能独立存在的。
- 一个Activity里可以有多个Fragment。
- 一个Fragment可以被多个Activity重用。
- Fragment有自己的生命周期，并能接收输入事件。
- 我们能在Activity运行时动态地添加或删除Fragment

## 基本使用
### 1.
这里给出Fragment最基本的使用方式。首先，创建继承Fragment的类，名为Fragment1：
```java
public class Fragment1 extends Fragment{  
  private static String ARG_PARAM = "param_key";
     private String mParam;
     private Activity mActivity;
     public void onAttach(Context context) {
        mActivity = (Activity) context;
        mParam = getArguments().getString(ARG_PARAM);  //获取参数
    }
     public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
        View root = inflater.inflate(R.layout.fragment_1, container, false);
        TextView view = root.findViewById(R.id.text);
        view.setText(mParam);
             return root;
    }    
     public static Fragment1 newInstance(String str) {
        Fragment1 frag = new Fragment1();
        Bundle bundle = new Bundle();
        bundle.putString(ARG_PARAM, str);
        fragment.setArguments(bundle);   //设置参数
        return fragment;
    }
}

```
Fragment有很多可以**复写**的方法，其中最常用的就是**onCreateView()** ，该方法返回Fragment的UI布局，需要注意的是inflate()的第三个参数是**false** ，因为在Fragment内部实现中，会把该布局添加到container中，如果设为true，那么就会重复做两次添加，则会抛如下异常：

```java
Caused by: java.lang.IllegalStateException:
The specified child already has a parent.
You must call removeView() on the child's parent first.
```

如果在创建Fragment时要传入参数，必须要通过`setArguments(Bundle bundle)`方式添加，而不建议通过为Fragment添加带参数的构造函数，因为通过`setArguments()`方式添加.在由于内存紧张导致Fragment被系统杀掉并恢复（re-instantiate）时能保留这些数据。官方建议如下：
```java
It is strongly recommended that subclasses do not have other constructors with parameters,
since these constructors will not be called when the fragment is re-instantiated.
```
我们可以在Fragment的`onAttach()`中通过`getArguments()`获得传进来的参数，并在之后使用这些参数。

### 2.
创建完Fragment后，接下来就是把Fragment添加到Activity中。在Activity中添加Fragment的方式有两种：
- 静态添加：通过xml的方式添加，缺点是一旦添加就不能在运行时删除。
- 动态添加：运行时添加，这种方式比较灵活，因此建议使用这种方式。
虽然Fragment能在XML中添加，但是这只是一个语法糖而已，Fragment并不是一个View，而是和Activity同一层次的。

这里只给出动态添加的方式。首先Activity需要有一个容器存放Fragment，一般是FrameLayout，因此在Activity的布局文件中加入FrameLayout：
```xml
<FrameLayout
    android:id="@+id/container"
    android:layout_width="match_parent"
    android:layout_height="match_parent"/>
```
然后在`onCreate()`中，通过以下代码将Fragment添加进Activity中。
```java
if (bundle == null) {
    getSupportFragmentManager().beginTransaction()
        .add(R.id.container, Fragment1.newInstance("hello world"), "f1")        //.addToBackStack("fname")
        .commit();
}
```
这里需要注意几点：

- 因为我们使用了support库的Fragment，因此需要使用getSupportFragmentManager()获取FragmentManager。

- `add()`是对Fragment众多操作中的一种，还有`remove()`, `replace()`等，第一个参数是根容器的id（FrameLayout的id，即”@id/container”），第二个参数是Fragment对象，第三个参数是fragment的tag名，指定tag的好处是后续我们可以通过`Fragment1 frag = getSupportFragmentManager().findFragmentByTag("f1")`从FragmentManager中查找Fragment对象。
在一次事务中，可以做多个操作，比如同时做`add().remove().replace()`。

- `commit()`操作是异步的，内部通过`mManager.enqueueAction()`加入处理队列。对应的同步方法为`commitNow()`，`commit()`内部会有checkStateLoss()操作，如果开发人员使用不当（比如`commit()`操作在onSaveInstanceState()之后），可能会抛出异常，而commitAllowingStateLoss()方法则是不会抛出异常版本的`commit()`方法，但是尽量使用`commit()`，而不要使用commitAllowingStateLoss()。

- `addToBackStack("fname")`是可选的。FragmentManager拥有回退栈（BackStack），类似于Activity的任务栈，如果添加了该语句，就把该事务加入回退栈，当用户点击返回按钮，会回退该事务（回退指的是如果事务是`add(frag1)`，那么回退操作就是`remove(frag1))`；如果没添加该语句，用户点击返回按钮会直接销毁Activity。
- Fragment有一个常见的问题，即Fragment重叠问题，这是由于Fragment被系统杀掉，并重新初始化时再次将fragment加入activity，因此通过在外围加if语句能判断此时是否是被系统杀掉并重新初始化的情况

## 生命周期
Fragment的生命周期和Activity类似，但比Activity的生命周期复杂一些，基本的生命周期方法如下图
![pic1](http://odsdowehg.bkt.clouddn.com/fragment_lifecycle.png)

- onAttach()：Fragment和Activity相关联时调用。可以通过该方法获取Activity引用，还可以通 -过getArguments()获取参数。
- onCreate()：Fragment被创建时调用。
- onCreateView()：创建Fragment的布局。
- onActivityCreated()：当Activity完成onCreate()时调用。
- onStart()：当Fragment可见时调用。
- onResume()：当Fragment可见且可交互时调用。
- onPause()：当Fragment不可交互但可见时调用。
- onStop()：当Fragment不可见时调用。
- onDestroyView()：当Fragment的UI从视图结构中移除时调用。
- onDestroy()：销毁Fragment时调用。
- onDetach()：当Fragment和Activity解除关联时调用。

上面的方法中，只有onCreateView()在重写时不用写super方法，其他都需要。

因为Fragment是依赖Activity的，因此为了讲解Fragment的生命周期，需要和Activity的生命周期方法一起讲，即Fragment的各个生命周期方法和Activity的各个生命周期方法的关系和顺序:
![pic2](https://upload-images.jianshu.io/upload_images/2952813-0f4f821975d72317.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/640/format/webp)

FragmentTransaction有一些基本方法，下面给出调用这些方法时，Fragment生命周期的变化：

- add(): onAttach()->…->onResume()。
- remove(): onPause()->…->onDetach()。
- replace(): 相当于旧Fragment调用remove()，新Fragment调用add()。
- show(): 不调用任何生命周期方法，调用该方法的前提是要显示的 Fragment已经被添加到容器，只是纯粹把Fragment UI的setVisibility为true。
- hide(): 不调用任何生命周期方法，调用该方法的前提是要显示的Fragment已经被添加到容器，只是纯粹把Fragment UI的setVisibility为false。
- detach(): onPause()->onStop()->onDestroyView()。UI从布局中移除，但是仍然被FragmentManager管理。
- attach(): onCreateView()->onStart()->onResume()。

## Fragment的通信

待续

## GuideFragment

网易云音乐项目中的引导界面用于viewPager的Fragment， 由于不止一个地方需要使用Fragment, 所以包装了一个基类 `BaseFragment`, 用于实现都需要的一些操作， 比如查找Fragment中的控件，设置数据和监听器等等。

使用自定义的findViewById在Fragment中找控件， 其中`getView()`是在OnCreateView中返回的View， 所以`init()`需要在`onCreateView`之后才能调用，最好在`OnViewCreated()`方法中。
```java
public class BaseFragment extends Fragment {
    /**
     * 找控件
     */
    protected void initViews() {

    }

    /**
     *设置数据
     */
    protected void initDatas() {
    }

    /**
     * 绑定监听器
     */
    protected void initListener() {
    }

    @Nullable
    public final <T extends View> T findViewById(@IdRes int id) {
        return getView().findViewById(id);
    }
    //使用第三方库glide中的时候需要传递Activity.
    public BaseActivity getMainActivity() {
        return (BaseActivity) getActivity();
    }

    void inits() {
        initViews();
        initDatas();
        initListener();
    }
}
//需要使用数据库的一些操作的Fragment
public class BaseCommanFragment extends BaseFragment {
    SharedPreferencesUtil sp;

    @Override
    protected void initDatas() {
        super.initDatas();
        sp = SharedPreferencesUtil.getInstance(getActivity().getApplicationContext());
    }
}
```

`GuideFragment.java`

在创建一个Fragment的时候需要动态的传入数据，所以在`newInstance()`方法中传入图片的ID， 然后再通过`setArguments()`方法来将数据保存。再在`initDatas`的时候获取到图片的ID， 通过第三方库Glide和自己包装的图片显示类`ImageUtil`显示。

**这样做的原因： 通过Adapter的`setDatas()` 方法设置显示的图片的数据，然后在`getItem()`需要返回一个显示图片的Fragment的时候通过`GuideFragment.newInstance`方法返回一个。**

```java
public class GuideFragment extends BaseCommanFragment {
    ImageView iv;

    private Integer imageId;
    private View view;


    public static GuideFragment newInstance(int imageId) {

        Bundle args = new Bundle();
        args.putSerializable(Consts.ID, imageId);
        GuideFragment fragment = new GuideFragment();
        fragment.setArguments(args);
        return fragment;
    }

    @Nullable
    @Override
    public View onCreateView(LayoutInflater inflater, @Nullable ViewGroup container, Bundle savedInstanceState) {
        view = inflater.inflate(R.layout.fragment_guide, container, false);
        return view;
    }

    @Override
    public void onViewCreated(@NonNull View view, @Nullable Bundle savedInstanceState) {
        super.onViewCreated(view, savedInstanceState);
        inits();
    }

    @Override
    protected void initViews() {
        super.initViews();
        iv= findViewById(R.id.iv);
    }
    @Override
    protected void initDatas() {
        super.initDatas();
        imageId = getArguments().getInt(Consts.ID, -1);

        if (imageId == -1) {
            LogUtil.w("Image id can not be empty!");
            getMainActivity().finish();
            return;
        }


        ImageUtil.showLocalImage(getMainActivity(),iv,imageId);

    }
}
```
