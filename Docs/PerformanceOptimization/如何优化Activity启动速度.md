在一些低配手机上，我们经常会遇到白屏或者页面切换卡顿等现象，这些都是 Activity 启动速度慢的表现，本文将分享一些小米系统开发过程中优化 Activity 启动速度的经验。

我们把优化分为两类，分别叫做硬优化和软优化，先来说说软优化。

> 软优化:
是指通过一些特殊的方式来达到优化的效果，这种方法一般会尽可能地少修改原代码的结构和逻辑。

一般主要用以下几种软优化的方法:

1. 预加载
2. 异步加载:费时的非 UI 操作放到另一个 Thread
3. 延迟加载:可以延迟展示的 UI 操作放到 IdleHandler 中

## 延迟加载

```java

publicclassBaseActivityextendsActivity { 
    private boolean mDelayFinished = false;

@Override
protected void onCreate(final Bundle savedInstanceState) {
        super.onCreate (savedInstanceState); 
        mDelayFinished = false;

        Looper.myQueue().addIdleHandler(new IdleHandler() { 
            @Override
            public boolean queueIdle() {
                onCreateDelay(savedInstanceState); 
                onStartDelay() ;
                onResumeDelay();
                mDelayFinished = true;
                return false; 
            }
    });
}

protected void onCreateDelay(Bundle bundle) {
    // TODO : 可以延迟展示的 UI 操作 
}
protected void onStartDelay() { }
protected void onResumeDelay() { }
protected void onPauseDelay() { }
protected void onStopDelay() { }
protected void onDestroyDelay() { }

@Override
protected void onResume() {
    super.onResume();
    if (mDelayFinished) {
        onResumeDelay(); 
    }
} 
@Override
protected void onPause() { 
    super.onPause();
    if (mDelayFinished) { 
        onPauseDelay();
    } 
}
@Override
protected void onStop() {
    super.onStop () ;
    if (mDelayFinished) {
        onStopDelay() ; 
    }
} 
@Override
protected void onDestroy() { 
    super.onDestroy();
    if (mDelayFinished) { 
        onDestroyDelay();
    } 
}
}
```


Looper.myQueue().addIdleHandler 能够保证操作在 Activity 启动完成之后再执
行，子类只要把可以延迟加载的 UI 操作放到对应的*Delay()即可。

> 硬优化

深究代码的细节，通过查看 trace 文件，一步步地优化到极致。

1. 去除耗时的图片，数据库等操作
2. 精简布局，减少层次，尽可能多地使用 ViewStub
3. 一些费时的初始化操作提前到进程初始化执行
4. 优化主题加载资源的速度
5. 优化 Activity.installDecor 速度
6. 一些方法的优化，例如 Long.valueof，spareArray，具体有更好的性能 
7.  调整切换动画效果

硬优化需要有一定耐心，一步步找到问题的根源，这种方法也是我们更主张的优化
方式，这种优化就需要对原代码的结构和逻辑做比较大改动。