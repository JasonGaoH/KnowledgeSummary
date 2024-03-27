> 文章已同步发表于微信公众号JasonGaoH，[仿京东、淘宝首页，通过两层嵌套的RecyclerView实现tab的吸顶效果](https://mp.weixin.qq.com/s?__biz=MzUyNTE2OTAzMQ==&mid=2247483770&idx=1&sn=38b95c3cff51e216248c0eb860155437&chksm=fa237992cd54f084802be4b0be2b52e4f5052019d51562f65245ca8b0faf3e0d61d296fc0e7d&token=1938879438&lang=zh_CN#rd)

### 为什么会有这篇文章

之前写过一篇文章[使用CoordinatorLayout过程中遇到的两个问题以及浅析CoordinatorLayout工作机制](https://juejin.im/post/5d233cc86fb9a07ec42b7f57)，这篇文章上主要讲了通过CoordinatorLayout实现tab吸顶的效果时遇到的问题，效果跟京东、淘宝首页类似，只不过实现方法不同而已，但是使用CoordinatorLayout来实现是会有不少细节问题是很难处理好的，下面会详细介绍。

首先我们可以简单看下京东首页的效果gif，来看看我们到底是要实现什么样的效果：

![image](https://raw.githubusercontent.com/JasonGaoH/Images/master/jingdong.gif)

京东首页的tab筛选区将feed分为两个部分，上面是各种不同item，tab的下半部分可以左右横滑，并且下拉可以加重更多，只要网络有数据的情况下理论上是可以无限下拉的。

其实用CoordinatorLayout来实现tab吸顶，如果能将一些细节问题处理好的话，其实大致可以实现类似京东首页的这个效果，具体细节问题可以参考文章开头说的之前的文章，文章里讲了下使用CoordinatorLayout来实现类似效果遇到的动画抖动问题以及页面回弹问题以及对应的解决方法。

那么为什么会不采用CoordinatorLayout来实现，转而采用嵌套RecyclerView的方式呢？

首先我们来看下CoordinatorLayout实现的大致布局：

![image](https://raw.githubusercontent.com/JasonGaoH/Images/master/CoordinatorLayout.png)

一个问题是从AppBarLayout滑动效果是不能传递到下面的ViewPager里去的，我尝试了各种方式都没能解决掉这个问题，可以简单看下Demo效果图：

![image](https://raw.githubusercontent.com/JasonGaoH/Images/master/coordinatorlayout_fix.gif)

从gif图大致可以看到AppBarLayout滑上去之后惯性消失了，tab下面的区域是不能接着滚动的。

> 这个惯性消失的问题，我在网上找到了一个一篇解决惯性消失的文章如下[支付宝首页交互三部曲3实现支付宝首页交互](https://blog.kyleduo.com/2017/07/21/alipay-home-3-alipay-home/),实现方式大致是自己把CoordinatorLayout这套机制再实现了一遍，因为是自己实现的，里面的一些机制是比较方便改动的，它处理惯性这个问题的逻辑大致是将AppBarLayout中未消费的y轴偏移量拿出来再交由RecyclerView去滑动，代码如下：

```java
mHeaderView.setOnHeaderFlingUnConsumedListener(new APHeaderView.OnHeaderFlingUnConsumedListener() {
    @Override
    public int onFlingUnConsumed(APHeaderView header, int targetOffset, int unconsumed) {
        APHeaderView.Behavior behavior = mHeaderView.getBehavior();
        int dy = -unconsumed;
        if (behavior != null) {
            mRecyclerView.scrollBy(0, dy);
        }
        return dy;
    }
});
```
> 这里由于篇幅原因，就不展开详细介绍了，感兴趣地同学可以点开上面的链接去研究研究，文章中也贴出了GitHub地址。

我们重新回到我们一开始的问题，为什么想替换掉CoordinatorLayout，另外一个问题是CoordinatorLayout这种实现相对比较简单，但是会导致页面的嵌套层级很深，我们从上面贴出来的布局来看，view嵌套的层级特别深，而且如果我们要实现类似京东或者淘宝首页这样的效果，在TabLayout上面的区域，也就是下图箭头标注的地方必须要采用RecyclerView的来实现，因为tab上半部分的内容和个数都是不确定的，使用RecyclerView才比较方便，但是这样页面的层级就更深了，加载速度也变得更慢了。

![image](https://raw.githubusercontent.com/JasonGaoH/Images/master/CoordinatorLayout_fix.png)

### 如何实现

要抛弃CoordinatorLayout，那么如何实现呢？

我们用Android Studio中的Layout Inspector工具看了下京东首页的布局，发现的确是采用两层RecyclerView嵌套来实现的，展示的布局大致如下所示：

![image](https://raw.githubusercontent.com/JasonGaoH/Images/master/jingdong_layout.png)

那么接下来是怎么去实现这个效果了。其实一开始我以为要采用嵌套滚动这套机制来实现，后来发现不采用嵌套滚动机制也是可以实现的。

现在我们可以大致构造这样一个布局：

![image](https://raw.githubusercontent.com/JasonGaoH/Images/master/nested_recycler_view.png)

我们把ViewPager以及TabLayout这一块作为外部RecyclerView的一个item，ViewPager可能会有个多个内部RecyclerView，只要我们能让外部RecyclerView和内部RecyclerView的滑动事件正确分发基本就可以解决这个问题了。

如果只是构造出这个布局出来，我们发现内部的RecyclerView都不会显示出来，因为滑动完全由外部RecyclerView接管了。

那么重点来了，这种情况如何处理？

其实RecyclerView的LayoutManager中有这两个方法用于判断RecyclerView在水平方向上和竖直方向上是否可以滚动的。

```
public boolean canScrollHorizontally() {
    return false;
}

public boolean canScrollVertically() {
    return false;
}
```
然后LayoutManager有各种不同的实现LinearLayoutManager，StaggeredGridLayoutManager等

这个LayoutManager中的canScrollVertically和canScrollHorizontally在RecyclerView的onInterceptTouchEvent中是会拿来作判断，判断当前RecyclerView是否需要处理滑动事件的。

> 还有一点需要注意：我们处理内外两层RecyclerView的滑动冲突问题，主要是想解决下面两种场景：一、手指上滑时，当外部RecyclerView滑动底部的时候，内部的RecyclerView能继续去响应用户的滑动，因为内部的RecyclerView理论上是可以无限滚动的；二、手指下滑时，当内部的RecyclerView滑动到顶部的时候，外部的RecyclerView能够继续响应用户的下滑事件。

其实上面已经说出了如何处理嵌套RecyclerView的最重要的点，其他的部分相当于都是一些细节的处理了。

在外部RecyclerView(下面成ParentRecyclerView)中的重写LayoutManager的canScrollVertically方法如下：

```
fun initLayoutManager() {
    val linearLayoutManager = object :LinearLayoutManager(context) {

        override fun canScrollVertically():Boolean {
            //找到当前的childRecyclerView
            val childRecyclerView = findNestedScrollingChildRecyclerView()
            //只有当前childRecyclerView滑动到顶部才认为ParentRecyclerView是可以竖直方向是可以滚动的
            return childRecyclerView == null || childRecyclerView.isScrollTop()
        }

    }
    linearLayoutManager.orientation = LinearLayoutManager.VERTICAL
    layoutManager = linearLayoutManager
}
```
在内部RecyclerView(以下称ChildRecyclerView)中定义了isScrollTop()，用于判断ChildRecyclerView是否滚动到顶部。
```
fun isScrollTop(): Boolean {
    //RecyclerView.canScrollVertically(-1)的值表示是否能向下滚动，false表示已经滚动到顶部
    return !canScrollVertically(-1)
}
```

另外，在ParentRecyclerView的onTouchEvent方法中：
```
override fun onTouchEvent(e: MotionEvent): Boolean {
        if(lastY == 0f) {
            lastY = e.y
        }
        if(isScrollEnd()) {
            //如果父RecyclerView已经滑动到底部，需要让子RecyclerView滑动剩余的距离
            val childRecyclerView = findNestedScrollingChildRecyclerView()
            childRecyclerView?.run {
                val deltaY = (lastY - e.y).toInt()
                if(deltaY != 0) {
                    scrollBy(0,deltaY)
                }
            }
        }
        lastY = e.y
        return try {
            super.onTouchEvent(e)
        } catch (e: Exception) {
            e.printStackTrace()
            false
        }
    }
```

关于滑动事件主要代码就是上面这些，具体可以可以看看项目代码[NestedRecyclerView](https://github.com/JasonGaoH/NestedRecyclerView)。

还有关于RecyclerView的fling部分，在RecyclerView的onScrollStateChanged回调中监听y轴的总的偏移量totalDy，然后在RecyclerView不滚动的时候交由内部或者外部RecyclerView去fling，这里就不赘述了，具体可以看项目的代码。

最后贴上项目的运行gif图展示：

![image](https://raw.githubusercontent.com/JasonGaoH/NestedRecyclerView/master/gif/nested_recyclerview_1.gif)

![image](https://raw.githubusercontent.com/JasonGaoH/NestedRecyclerView/master/gif/nested_recyclerview_2.gif)

### 最后
写作不易，欢迎大家点赞，如果有问题，欢迎提出一起讨论，您的点赞是我写作的最大动力，感谢！