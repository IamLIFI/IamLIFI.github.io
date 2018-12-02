---
layout:     post                    # 使用的布局（不需要改）
title:      ViewPager               # 标题
subtitle:   ViewPager 和 Adapter #副标题
date:       2018-12-2           # 时间
author:     IamLIFI                     # 作者
header-img: img/post-bg-2015.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - Android
    - 仿网易云音乐
---

> http://www.apkbus.com/android-90417-1-1.html

## ViewPager

ViewPager 如其名所述，是负责翻页的一个 View。准确说是一个 ViewGroup，包含多个 View 页，在手指横向滑动屏幕时，其负责对 View 进行切换。为了生成这些 View 页，需要提供一个 PagerAdapter 来进行和数据绑定以及生成最终的 View 页。

- setAdapter()

  - ViewPager 通过 setAdapter() 来建立与 PagerAdapter 的联系。<br>这个联系是双向的，<br>一方面，ViewPager 会拥有 PagerAdapter 对象，从而可以在需要时调用 PagerAdapter 的方法；<br>另一方面，ViewPager 会在 setAdapter() 中调用 PagerAdapter 的 registerDataSetObserver() 方法，注册一个自己生成的 PagerObserver 对象，从而在 PagerAdapter 有所需要时（如 notifyDataSetChanged()或 notifyDataSetInvalidated() 时），可以调用 Observer 的 onChanged() 或 onInvalidated() 方法，从而实现 PagerAdapter 向 ViewPager 方向发送信息。

- dataSetChanged()

  - 在 PagerObserver.onChanged()，以及 PagerObserver.onInvalide() 中被调用。因此当 PagerAdapter.notifyDataSetChanged() 被触发时，ViewPager.dataSetChanged() 也可以被触发。该函数将使用 getItemPosition() 的返回值来进行判断，如果为 POSITION_UNCHANGED，则什么都不做；如果为 POSITION_NONE，则调用 PagerAdapter.destroyItem() 来去掉该对象，并设置为需要刷新 (needPopulate = true) 以便触发 PagerAdapter.instantiateItem() 来生成新的对象。

## PagerAdapter

PageAdapter 是 ViewPager 的支持者，ViewPager 将调用它来取得所需显示的页，而 PageAdapter 也会在数据变化时，通知 ViewPager。这个类也是FragmentPagerAdapter 以及 FragmentStatePagerAdapter 的基类。如果继承自该类，至少需要实现 **instantiateItem(), destroyItem(), getCount() 以及 isViewFromObject()**

- getItemPosition()                                 
  - 该函数用以返回给定对象的位置，给定对象是由 instantiateItem() 的返回值。
  - 在 ViewPager.dataSetChanged() 中将对该函数的返回值进行判断，以决定是否最终触发 PagerAdapter.instantiateItem() 函数。
  - 在 PagerAdapter 中的实现是直接传回 POSITION_UNCHANGED。如果该函数不被重载，则会一直返回 POSITION_UNCHANGED，从而导致 ViewPager.dataSetChanged() 被调用时，认为不必触发 PagerAdapter.instantiateItem()。很多人因为没有重载该函数，而导致调用PagerAdapter.notifyDataSetChanged() 后，什么都没有发生。

- instantiateItem()
  - 在每次 ViewPager 需要一个用以显示的 Object 的时候，该函数都会被 ViewPager.addNewItem() 调用。

- notifyDataSetChanged()
  - 在数据集发生变化的时候，一般 Activity 会调用 PagerAdapter.notifyDataSetChanged()，以通知 PagerAdapter，而 PagerAdapter 则会通知在自己这里注册过的所有 DataSetObserver。其中之一就是在 ViewPager.setAdapter() 中注册过的 PageObserver。PageObserver 则进而调用 ViewPager.dataSetChanged()，从而导致 ViewPager 开始触发更新其内含 View 的操作。

## FragmentPagerAdapter
FragmentPagerAdapter 继承自 PagerAdapter。相比通用的 PagerAdapter，该类更专注于每一页均为 Fragment 的情况。如文档所述，该类内的每一个生成的 Fragment 都将保存在**内存**之中，因此适用于那些**相对静态的页，数量也比较少**的那种；如果需要处理有很多页，并且数据动态性较大、占用内存较多的情况，应该使用**FragmentStatePagerAdapter**。<br>FragmentPagerAdapter 重载实现了几个必须的函数，因此来自 PagerAdapter 的函数，我们只需要实现 **getCount()** 即可。且，由于 FragmentPagerAdapter.instantiateItem() 的实现中，调用了一个新增的虚函数 getItem()，因此，我们还至少需要实现一个 **getItem()**。因此，总体上来说，相对于继承自 PagerAdapter，更方便一些.
- getItem()
  - 该类中新增的一个虚函数。函数的目的为生成新的 Fragment 对象。重载该函数时需要注意这一点。在需要时，该函数将被 instantiateItem() 所调用。
  - 如果需要向 Fragment 对象传递相对**静态的数据**时，我们一般通过 **Fragment.setArguments()** 来进行，这部分代码应当放到 getItem()。它们只会在新生成 Fragment 对象时**执行一遍**。
  - 如果需要在生成 Fragment 对象**后**，将数据集里面一些**动态**的数据传递给该 Fragment，那么，这部分代码**不适合**放到 getItem() 中。因为当数据集发生变化时，往往对应的 Fragment 已经生成，如果传递数据部分代码放到了 getItem() 中，这部分代码将不会被调用。这也是为什么很多人发现调用 PagerAdapter.notifyDataSetChanged() 后，getItem() 没有被调用的一个原因。(getItem 被instantiateItem方法调用生成显示一个Object)
- instantiateItem()
    - 函数中判断一下要生成的 Fragment 是否已经生成过了，如果生成过了，就使用旧的，旧的将被 Fragment.attach()；如果没有，就调用 getItem() 生成一个新的，新的对象将被 FragmentTransation.add()。
    - FragmentPagerAdapter 会将所有生成的 Fragment 对象通过 FragmentManager 保存起来备用，以后需要该 Fragment 时，都会从 FragmentManager 读取，而**不会**再次调用 getItem() 方法。
    - 如果需要在生成 Fragment 对象后，将数据集中的一些数据传递给该 Fragment，这部分代码应该放到这个函数的重载里。在我们继承的子类中，重载该函数，并调用 FragmentPagerAdapter.instantiateItem() 取得该函数返回 Fragment 对象，然后，我们该 Fragment 对象中对应的方法，将数据传递过去，然后返回该对象。
    - 否则，如果将这部分传递数据的代码放到 getItem()中，在 PagerAdapter.notifyDataSetChanged() 后，这部分数据设置代码将不会被调用。
- destroyItem()
    - 该函数被调用后，会对 Fragment 进行 FragmentTransaction.detach()。这里不是 remove()，只是 detach()，因此 Fragment 还在 FragmentManager 管理中，Fragment 所占用的资源不会被释放。

## 仿网易云项目中使用ViewPager和FragmentPagerAdapter来实现引导页面的滑动


### BaseFragmentPagerAdapter
由于项目别的地方也需要使用PagerAdapter, 而这些Adapter可能和GuideActivty的adapter不一样， 所以抽象了一层包装成基类。
```java
public abstract class BaseFragmentPagerAdapter<T> extends FragmentPagerAdapter {
    protected final Context context;
    protected final List<T> datas = new ArrayList<T>();

    //  FragmentManager 保存已生成的那些fragment
    public BaseFragmentPagerAdapter(Context context, FragmentManager fm) {
        super(fm);
        this.context = context;
    }
    // Adapter 都需要设置和获取他们用来显示的datas
    public T getData(int position) {
        return datas.get(position);
    }

    public void setDatas(List<T> data) {
        if (data != null && data.size() > 0) {
            datas.clear();
            datas.addAll(data);
            notifyDataSetChanged();
        }
    }

    public void addDatas(List<T> data) {
        if (data != null && data.size() > 0) {
            datas.addAll(data);
            notifyDataSetChanged();
        }
    }
    // 继承自 FragmentPagerAdapter需要重载的一个方法
    @Override
    public int getCount() {
        return datas.size();
    }
}

```

### GuideAdapter
```java
public class GuideAdapter extends BaseFragmentPagerAdapter<Integer>{

    public GuideAdapter(Context context, FragmentManager fm) {
        super(context, fm);
    }
//需要重载的第二个方法
    @Override
    public Fragment getItem(int position) {
        return GuideFragment.newInstance(getData(position));
    }
}
```

## 用到的自己包装的类
[GuideFragment](https://baidu.com)
