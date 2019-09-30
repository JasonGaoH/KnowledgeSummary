在使用CoordinatorLayout来实现Android中的一种吸顶的时候，遇到了两个CoordinatorLayout的滑动问题，这里做下记录。

这里使用CoordinatorLayout实现的是一个tab吸顶的效果，类似淘宝，京东首页的一个效果。
头部区域展示各种类型banner卡片，中间是类似TabLayout的可点击tab，下面是feed卡片，可以一直下拉加载，并且feed卡片区域使用ViewPager可以支持左右横滑切换tab，另外，就是tab滚动到顶部之后会有个吸顶的效果。

我们在项目中也要实现的效果，一开始我的想法是使用嵌套RecycleView的形式来实现，因为我去调研了下京东和淘宝的首页布局都是这么实现的，京东和淘宝首页实现方式和下面的图类似，外部的整个RecycleView嵌套ViewPager，ViewPager中再有多个RecycleView，这个实现起来稍微有点麻烦，难点是要处理好外部的RecycleView和ViewPager中内部RecycleView的滑动事件传递，这里我们只是简单介绍下，后面我会专门来介绍类似这样的嵌套RecycleView如何实现。

![image](https://raw.githubusercontent.com/JasonGaoH/Images/master/nested_recycler_view.png)

接下来是如何采用其他方便的方式来实现类似需求？我想到了CoordinatorLayout，CoordinatorLayout在处理吸顶是有一套已经成熟的方案的。

网上关于CoordinatorLayout的使用有很多不错的文章，这里就不介绍如何使用，关于CoordinatorLayout和Behavior我推荐看看这篇文章[针对 CoordinatorLayout 及 Behavior 的一次细节较真](https://blog.csdn.net/briblue/article/details/73076458)

而我们这篇文章主要是讲使用CoordinatorLayout中遇到的问题，问题如何解决以及CoordinatorLayout为什么会有这样的问题。

![image](https://raw.githubusercontent.com/JasonGaoH/Images/master/CoordinatorLayout.png)

实现这个大概是像上面这样类似的布局结构，来看下布局文件。

```
<?xml version="1.0" encoding="utf-8"?>
<android.support.design.widget.CoordinatorLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <android.support.design.widget.AppBarLayout
        app:layout_scrollFlags="scroll"
        android:layout_width="match_parent"
        android:layout_height="wrap_content">

        <android.support.design.widget.CollapsingToolbarLayout
            app:layout_scrollFlags="scroll"
            app:scrimVisibleHeightTrigger="45dp"
            android:layout_width="match_parent"
            android:layout_height="wrap_content">

            <ImageView
                android:background="@drawable/header"
                app:layout_scrollFlags="scroll"
                android:layout_width="match_parent"
                android:layout_height="450dp" />

        </android.support.design.widget.CollapsingToolbarLayout>

        <android.support.design.widget.TabLayout
            android:id="@+id/tabs"
            app:layout_collapseMode="pin"
            app:tabMode="scrollable"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"/>

    </android.support.design.widget.AppBarLayout>


    <android.support.v4.view.ViewPager
        android:id="@+id/view_pager"
        app:layout_behavior="@string/appbar_scrolling_view_behavior"
        android:layout_width="match_parent"
        android:layout_height="match_parent"/>

</android.support.design.widget.CoordinatorLayout>
```

这样的布局，接着填充数据基本上就能实现tab吸顶效果，feed卡片区采用RecycleView实现，可以一直下拉，并且能够支持左右横滑，基本实现了类似京东，淘宝首页的一个效果。

但是在使用这种方式来实现发现两个很明显的问题。

#### 第一，抖动问题
该问题场景描述：我们触摸AppBarLayout使AppBarLayout整体向上滑动，，即手指上滑，当AppBarLayout fling的同时，我们触摸下部ViewPager中的RecycleView区域，使RecycleView区域整体向下滑动，即手指下滑，这个时候会发现一个明显页面动画现象，这个问题几乎是必现。

来看下gif效果：![image](https://raw.githubusercontent.com/JasonGaoH/Images/master/coordinator_resize_1.gif)

接下来我们来看问题的原因，其实这个要搞清楚原因需要对CoordinatorLayout的工作机制有个比较清晰的理解，然而CoordinatorLayout这里牵扯到嵌套滚动以及Behavior这些，

我们这里尝试简单地介绍下CoordinatorLayout的工作机制。

![image](https://raw.githubusercontent.com/JasonGaoH/Images/master/coordinatorLayout_analysis.png)


- CoordinatorLayout实现NestedScrollingParent2接口，用于处理与滑动子View的联动交互，实际上交由Behavior进行处理。
- AppBarLayout中默认使用了AppBarLayout.Behavior，主要功能是接收CoordinatorLayout传输过来的滑动事件，并且相对应的进行处理，如RecycleView往上滑动到头时候，继续滑动移动AppBarLayout到头。
- RecycleView实现了NestedScrollingChild2接口，用于传输给CoordinatorLayout，并且消费CoordinatorLayout不消费的触摸事件，其中还是使用了AppBarLayout.ScrollingViewBehavior，功能是进行监听AppBarLayout的位移变化，从而进行相对应的变化，最明显的例子就是AppBarLayout上移过程中，RecycleView一起上移。

#### CoordinatorLayout中Behavior
其实CoordinatorLayout就是通过Behavior这个机制来协调各个子View的滚动。比如我们来看CoordinatorLayout的onStartNestedScroll方法，这个其实是NestedScrollingParent2中的方法。

当CoordinatorLayout子view的调用NestedScrollingChild2的方法startNestedScroll时,会调用到该方法
该方法决定了当前控件是否能接收到其内部View（并非是直接子View）滑动时的参数。

```
//CoordinatorLayout中的onStartNestedScroll方法：
 @Override
    public boolean onStartNestedScroll(View child, View target, int nestedScrollAxes) {
        return onStartNestedScroll(child, target, nestedScrollAxes, ViewCompat.TYPE_TOUCH);
    }

    @Override
    public boolean onStartNestedScroll(View child, View target, int axes, int type) {
        boolean handled = false;

        final int childCount = getChildCount();
        for (int i = 0; i < childCount; i++) {
            final View view = getChildAt(i);
            if (view.getVisibility() == View.GONE) {
                // If it's GONE, don't dispatch
                continue;
            }
            final LayoutParams lp = (LayoutParams) view.getLayoutParams();
            final Behavior viewBehavior = lp.getBehavior();
            if (viewBehavior != null) {
                final boolean accepted = viewBehavior.onStartNestedScroll(this, view, child,
                        target, axes, type);
                handled |= accepted;
                lp.setNestedScrollAccepted(type, accepted);
            } else {
                lp.setNestedScrollAccepted(type, false);
            }
        }
        return handled;
    }
```
CoordinatorLayout中的onStartNestedScroll方法基本都会调用到每个子View的Behavior中相应的方法中去。

关于Nested嵌套滚动机制可以看看下面这篇博客。

[事件分发和NestedScrolling](https://blog.csdn.net/aishang5wpj/article/details/78873343)

嵌套滚动机制NestedScrollingParent2和NestedScrollingChild2的各个回调方法调用流程如下图所示：
![image](https://raw.githubusercontent.com/JasonGaoH/Images/master/nested.jpeg)

上图列出来手指从按下到抬起时的整个流程，当然这些都是在子View的onTouchEvent()中完成的，所以父View一定不能拦截子View的事件，否则这套机制就失效了。

除此之外，箭头的左边分别都是NestedScrollingChild2中的各种方法，右边则是NestedScrollingParent2对应的方法。使用时，一般是子View通过dispatchXXX()来通知父View，然后父View通过onXXX()来进行回应。

方法调用的先后时机也有区别，对应到上图中，图越往下，调用的时机越晚。

#### AppBarLayout中的Behavior

接着我们来看看AppBarLayout中的Behavior，ApprBarLayout的默认Behavior就是AppBarLayout.Behavior这个类，而AppBarLayout.Behavior继承自HeaderBehavior，HeaderBehavior又继承自ViewOffsetBehavior，这里先总结一下两个类的作用。
- ViewOffsetBehavior：该Behavior主要运用于View的移动，从名字就可以看出来，该类中提供了上下移动，左右移动的方法。
- HeaderBehavior：该类主要用于View处理触摸事件以及触摸后的fling事件。


由于上面两个类功能的实现，使得AppBarLayout.Behavior具有了同时移动本身以及处理触摸事件的功能。

```
public boolean onTouchEvent(CoordinatorLayout parent, V child, MotionEvent ev) {
       ...
        switch (ev.getActionMasked()) {
            ...
            case MotionEvent.ACTION_MOVE: {
                final int activePointerIndex = ev.findPointerIndex(mActivePointerId);
                if (activePointerIndex == -1) {
                    return false;
                }

                final int y = (int) ev.getY(activePointerIndex);
                int dy = mLastMotionY - y;

                if (!mIsBeingDragged && Math.abs(dy) > mTouchSlop) {
                    mIsBeingDragged = true;
                    if (dy > 0) {
                        dy -= mTouchSlop;
                    } else {
                        dy += mTouchSlop;
                    }
                }

                if (mIsBeingDragged) {
                    mLastMotionY = y;
                    // We're being dragged so scroll the ABL
                    scroll(parent, child, dy, getMaxDragOffset(child), 0);
                }
                break;
            }

            case MotionEvent.ACTION_UP:
                if (mVelocityTracker != null) {
                    mVelocityTracker.addMovement(ev);
                    mVelocityTracker.computeCurrentVelocity(1000);
                    float yvel = mVelocityTracker.getYVelocity(mActivePointerId);
                    fling(parent, child, -getScrollRangeForDragFling(child), 0, yvel);
                }
         ...
        return true;
    }
```

我们来看onTouchEvent的方法，主要逻辑还是在ACTION_MOVE中，可以看到在滑动过程中调用了scroll(...)方法，scroll(...)方法在HeaderBehavior中进行实现，最终调用到了额setHeaderTopBottomOffset(...)方法，该方法在AppBarLayout.Behavior中进行了重写，所以，我们直接看AppBarLayout.Behavior中的源码即可：

```
@Override
   //newOffeset传入了dy，也就是我们手指移动距离上一次移动的距离，
   //minOffset等于AppBarLayout的负的height，maxOffset等于0。
        int setHeaderTopBottomOffset(CoordinatorLayout coordinatorLayout,
                AppBarLayout appBarLayout, int newOffset, int minOffset, int maxOffset) {
            final int curOffset = getTopBottomOffsetForScrollingSibling();//获取当前的滑动Offset
            int consumed = 0;
			//AppBarLayout滑动的距离如果超出了minOffset或者maxOffset，则直接返回0
            if (minOffset != 0 && curOffset >= minOffset && curOffset <= maxOffset) {
               //矫正newOffset，使其minOffset<=newOffset<=maxOffset
                newOffset = MathUtils.clamp(newOffset, minOffset, maxOffset);
				//由于默认没设置Interpolator，所以interpolatedOffset=newOffset;
                if (curOffset != newOffset) {
                    final int interpolatedOffset = appBarLayout.hasChildWithInterpolator()
                            ? interpolateOffset(appBarLayout, newOffset)
                            : newOffset;
					//调用ViewOffsetBehvaior的方法setTopAndBottomOffset(...)，最终通过
					//ViewCompat.offsetTopAndBottom()移动AppBarLayout
                    final boolean offsetChanged = setTopAndBottomOffset(interpolatedOffset);

                    //记录下消费了多少的dy。
                    consumed = curOffset - newOffset;
                   //没设置Interpolator的情况， mOffsetDelta永远=0
                    mOffsetDelta = newOffset - interpolatedOffset;
					....
                     //分发回调OnOffsetChangedListener.onOffsetChanged(...)
                    appBarLayout.dispatchOffsetUpdates(getTopAndBottomOffset());

                  
                    updateAppBarLayoutDrawableState(coordinatorLayout, appBarLayout, newOffset,
                            newOffset < curOffset ? -1 : 1, false);
                }
           ...
            return consumed;
        }
```
AppBarLayout中移动主要就是这部分逻辑了，通过setTopAndBottomOffset()来达到了移动我们的AppBarLayout，那么这里AppBarLayout就可以跟着手上下移动了。

#### RecycleView中的Behavior
那么接下来我们看看RecycleView在CoordinatorLayout中如何是移动的？

上面讲了AppBarLayout是如何通过Behavior来移动的，我们在上面布局文件中指定了ViewPager的Behavior。

```
app:layout_behavior="@string/appbar_scrolling_view_behavior"
```
这个"appbar_scrolling_view_behavior"其实就是ScrollingViewBehavior，ScrollingViewBehavior也继承自ViewOffsetBehavior，我们在上下移动AppBarLayout的时候，下面的RecycleView也是需要跟着移动的，它上下移动就是靠这个来ScrollingViewBehavior来实现的。

在阅读ScrollingViewBehavior源码中发现其实现了如下方法：
```
@Override
public boolean layoutDependsOn(CoordinatorLayout parent, View child, View dependency) {
    // We depend on any AppBarLayouts
    return dependency instanceof AppBarLayout;
}

@Override
public boolean onDependentViewChanged(CoordinatorLayout parent, View child,
        View dependency) {
    offsetChildAsNeeded(parent, child, dependency);
    return false;
}
```
这样我们这个RecycleView依赖于AppBarLayout，在AppBarLayout移动的过程中，RecycleView会随着AppBarLayout的移动回调onDependentViewChanged(...)方法，进而调用offsetChildAsNeeded(parent, child, dependency)。


用这么多篇幅主要讲了CoordinatorLayout如何协调AppBarLayout和RecycleView来上下滚动的，接着回到刚开始我们要讨论那个动画抖动问题。

其实造成这个的原因主要是AppBarLayout的fling操作和RecycleView联动造成的问题。

在AppBarLayout的Behavior中的onTouchEvent()事件中处理了fling事件：
```
public boolean onTouchEvent(CoordinatorLayout parent, V child, MotionEvent ev) {
    ...
    case MotionEvent.ACTION_UP:
        if (mVelocityTracker != null) {
            mVelocityTracker.addMovement(ev);
            mVelocityTracker.computeCurrentVelocity(1000);
            float yvel = mVelocityTracker.getYVelocity(mActivePointerId);
            fling(parent, child, -getScrollRangeForDragFling(child), 0, yvel);
        }
    ...
    return true;
}
```
在fling的方法中使用OverScroller来模拟进行fling操作，最终会调到setHeaderTopBottomOffset(...)来使AppBarLayout进行fling的滑动操作。

在绝大部分滑动逻辑中，这样处理是正确的，但是如果在AppBarLayout在fling的时候主动滑动RecyclerView，那么就会造成动画抖动的问题了。

在当前情况下，RecyclerView滑动到头了，那么就会把未消费的事件通过NestedScrollingChild2交付由CoordinatorLayout(实现了NestedScrollingParent2)处理，parent又最终交付由AppBarLayout.Behavior进行处理的，其中调用的方法如下：
```
@Override
public void onNestedScroll(CoordinatorLayout coordinatorLayout, AppBarLayout child,
        View target, int dxConsumed, int dyConsumed, int dxUnconsumed, int dyUnconsumed,
        int type) {
    if (dyUnconsumed < 0) {
        // If the scrolling view is scrolling down but not consuming, it's probably be at
        // the top of it's content
        scroll(coordinatorLayout, child, dyUnconsumed,
                -child.getDownNestedScrollRange(), 0);
    }
}
```

这里的scroll方法最终会调用setHeaderTopBottomOffset(...)，由于两次分别触摸在AppBarLayout和RecyclerView的方向不一致，导致了最终的抖动的效果。

解决方式也很简单,只要在CoordinatorLayout的onInterceptedTouchEvent()中停止AppBarLayout的fling操作就可以了，直接操作的对象就是AppBarLayout中的Behavior，该Behavior继承自HeaderBehavior，而fling操作由OverScroller产生，所以自定义一个FixedBehavior：
```
public class FixedBehavior extends AppBarLayout.Behavior {
    private OverScroller mOverScroller;

    public FixedBehavior() {
        super();
    }

    public FixedBehavior(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    @Override
    public void onAttachedToLayoutParams(@NonNull CoordinatorLayout.LayoutParams params) {
        super.onAttachedToLayoutParams(params);
    }

    @Override
    public void onDetachedFromLayoutParams() {
        super.onDetachedFromLayoutParams();
    }

    @Override
    public boolean onTouchEvent(CoordinatorLayout parent, AppBarLayout child, MotionEvent ev) {
        if (ev.getAction() == MotionEvent.ACTION_UP) {
            reflectOverScroller();
        }
        return super.onTouchEvent(parent, child, ev);
    }

    /**
     *
     */
    public void stopFling() {
        if (mOverScroller != null) {
            mOverScroller.abortAnimation();
        }
    }

    /**
     * 解决AppbarLayout在fling的时候，再主动滑动RecyclerView导致的动画错误的问题
     */
    private void reflectOverScroller() {
        if (mOverScroller == null) {
            Field field = null;
            try {
                field = getClass().getSuperclass()
                        .getSuperclass().getDeclaredField("mScroller");
                field.setAccessible(true);
                Object object = field.get(this);
                mOverScroller = (OverScroller) object;
            } catch (NoSuchFieldException e) {
                e.printStackTrace();
            } catch (IllegalAccessException e) {
                e.printStackTrace();
            }

        }
    }
}
```

然后在重写CoordinatorLayout，暴露一个接口：
```
public class CustomCoordinatorLayout extends CoordinatorLayout {
    private OnInterceptTouchListener mListener;

    public void setOnInterceptTouchListener(OnInterceptTouchListener listener) {
        mListener = listener;
    }

    public CustomCoordinatorLayout(Context context) {
        super(context);
    }

    public CustomCoordinatorLayout(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    public CustomCoordinatorLayout(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
    }

    @Override
    public boolean onInterceptTouchEvent(MotionEvent ev) {
        if (mListener != null) {
            mListener.onIntercept();
        }
        return super.onInterceptTouchEvent(ev);
    }


    public interface OnInterceptTouchListener {
        void onIntercept();
    }
}
```
接着在接口中处理滑动问题即可：
```
coordinatorLayout.setOnInterceptTouchListener {
    //RecyclerView滑动的时候禁止AppBarLayout的滑动
    if (customBehavior != null) {
        customBehavior!!.stopFling()
    }
}
```

#### 第二，回弹问题

问题场景描述，我们反复上下滑动AppBarLayout的时候，可以看到AppBarLayout在滑出屏幕外之后又反弹回去了，而且当你滑动的加速度很大的时候，这个反弹的幅度也会跟着变大。

![image](https://raw.githubusercontent.com/JasonGaoH/Images/master/coordinator_resize_2.gif)

这个问题造成的原因是因为在手指向上滑动后造成RecyclerView的fling操作执行，具体的代码在RecyclerView内部类ViewFlinger中。

我使用Android Studio中的Profiler抓取了一下当出现反弹问题的时候出现的方法调用堆栈如下所示：
![image](https://raw.githubusercontent.com/JasonGaoH/Images/master/coordinator_bugs_method_call.jpg)


发现RecyclerView中ViewFlinger调用后，接着触发了HeaderBehavior中的FlingRunnable。而ViewFling中会调用dispatchNestedScroll(...)方法，RecyclerView作为CoordinatorLayout的子View，它通过嵌套滚动的机制又会调用到CoordinatorLayout中的onNestedScroll，这里主要就是通过AppBarLayout的Behavior中的方法setHeaderTopBottomOffset来实现AppBarLayout的滚动，后面会发现多次setHeaderTopBottomOffset的调用，其实目前看到这里，并不太确定造成这个问题的具体原因是啥，感觉上是因为RecyclerView的滑动和CoordinatorLayout的滑动冲突导致了反弹效果的出现。


于是尝试了下面的解决方法：
```
coordinatorLayout.setOnInterceptTouchListener {
    mRecyclerView.stopScroll()
}
```

试了这个方法发现果然有效。


另外，我在写demo的时候发现，这个问题在support-27是存在的，在support-28 Google已经修复过了。

我尝试过看看support-28里面的都有哪些改动，想看看Google是如何修复的。看了下Google的release note并没有提及，如果从Google的commit history来看实在页看不出来啥，暂时也没有个具体的原因。

后面可以将support-27和support-28的source下载下来，然后使用Beyond Compare来看看具体的diff改动是在哪。


