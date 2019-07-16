## KnowledgeSummary目录：

* [Java](#Java)
* [Android](#Android)
* [开源库](#开源库)
* [性能优化](#性能优化)
* [图片相关](#图片相关)
* [Android其他](#Android其他)
* [多线程相关](#多线程相关)
* [网络](#网络)
* [设计模式](#设计模式)
* [算法](#算法)
* [操作系统](#操作系统)

### Java
* Java基础
  * [Java基础知识总结](./Docs/Java/Java基础知识总结.md)
  * [Java中的集合框架](./Docs/Java/Java中的集合框架.md)
  * [从源码角度分析ArrayList和Vector的区别](./Docs/Java/从源码角度分析ArrayList和Vector的区别.md)
  * [从源码角度分析ArrayList和LinkedList的区别](./Docs/Java/从源码角度分析ArrayList和LinkedList的区别.md)
  * [从源码角度分析HashMap的原理与实现](./Docs/Java/从源码角度分析HashMap的原理与实现.md)

* Java进阶
  * [Java中的注解](./Docs/Java/Java中的注解.md)
  * [Java中的ClassLoader机制](./Docs/Java/Java中的ClassLoader机制.md)
  * [可否用try-catch捕获OOM以避免其发生](./Docs/Java/可否用try-catch捕获OOM以避免其发生.md)

### Android
* UI
    * [SurfaceView VS TextureView](./Docs/Android/UI/SurfaceView和TextureView的区别.md)

* Activity和Fragment
  * [Activity的生命周期和启动模式](./Docs/Android/Activity/Activity的生命周期和启动模式.md)
  * [launchMode](./Docs/Android/其他/launchMode.md)
  * [Activity调用方式](./Docs/Android/其他/Activity调用方式.md)
  * [AlertDialog，Toast对Activity生命周期的影响](./Docs/Android/其他/AlertDialog，Toast对Activity生命周期的影响.md) 
  * [Activity之间的通信方式](/Docs/Android/其他/Activity之间的通信方式.md)
  * [Android里面为什么要设计出Bundle而不是直接用Map结构](https://github.com/android-cn/android-discuss/issues/142)

* Service
  * [Service的生命周期](./Docs/Android/Service/Service的生命周期.md)
  * 怎么启动service，service和activity怎么进行数据交互
  * [如何在后台下载任务, 并在通知栏显示进度](https://juejin.im/post/586072c861ff4b005820901d)

* View
  * [View事件传递](./Docs/Android/其他/Android中的事件传递.md)
  * [如何计算一个View的层级](./Docs/Android/UI/如何计算一个View的层级.md)
  * [TextView性能瓶颈，渲染优化，以及StaticLayout的一些用处](https://www.jianshu.com/p/9f7f9213bff8)

### 开源库

* [Fresco](./Docs/Android/开源库/Fresco.md)						
* [Tinker](./Docs/Android/开源库/tinker.md)						
* [ActivityRouter](./Docs/Android/开源库/ActivityRouter.md)				
* [ARouter](./Docs/Android/开源库/ARouter.md)					
* [EventBus](./Docs/Android/开源库/EventBus.md)						
* [RxJava](./Docs/Android/开源库/RxJava.md)						
* [Retrofit](./Docs/Android/开源库/Retrofit.md)						
* [OKHTTP](./Docs/Android/开源库/OKHTTP.md)						
* [LeakCanary](./Docs/Android/开源库/LeakCanary.md)					
* [Atlas](./Docs/Android/开源库/atlast.md)							
* [BlockCanary](./Docs/Android/开源库/BlockCanary.md)

### 性能优化

* [绘制优化](./Docs/PerformanceOptimization/绘制优化.md)
* [启动优化](./Docs/PerformanceOptimization/启动优化.md)	
* [内存优化](./Docs/PerformanceOptimization/内存优化.md)
* [稳定性优化](./Docs/PerformanceOptimization/稳定性优化.md)
* [ANR分析](./Docs/PerformanceOptimization/ANR分析.md)
* [耗电优化](./Docs/PerformanceOptimization/耗电优化.md)
* [安装包优化](./Docs/PerformanceOptimization/安装包优化.md)

### 图片相关

* [LruCache](./Docs/Android/图片相关/LruCache.md)
* [DiskLruCache](./Docs/Android/图片相关/DiskLruCache.md)
* [图像显示原理](./Docs/Android/图片相关/图像显示原理.md)
* [Bitmap和Drawable	](./Docs/Android/图片相关/bitmap_vs_drawable.md)
* [图片加载优化](./Docs/Android/图片相关/图片加载优化.md)

### Android其他

* [Art和Dalvik区别](./Docs/Android/其他/Art和Dalvik区别.md)
* [详解注解处理器APT技术](./Docs/Android/其他/详解APT.md)
* [APK打包及安装过程](./Docs/Android/其他/APK打包及安装过程.md)
* [IntentFilter匹配规则](./Docs/Android/其他/IntentFilter匹配规则.md)
* [插件化](./Docs/Android/其他/插件化.md)
* [组件化](./Docs/Android/其他/组件化.md)
* [热修复方案对比](./Docs/Android/其他/热修复.md)
* App启动流程，从点击桌面开始 [link](http://www.androidos.net.cn/doc/day/2018-02-18/15384.md)
* [数据库如何进行升级,SQLite增删改查的基础sql语句](./Docs/Android/其他/数据库如何进行升级,SQLite增删改查的基础sql语句.md)


### 多线程相关

* [死锁](./Docs/MultiThread/死锁.md)
* [Android多进程](./Docs/MultiThread/Android多进程.md)
* [Android进程间通信--Binder	](./Docs/MultiThread/Android进程间通信--Binder.md)
* [多线程间通信](./Docs/MultiThread/多线程间通信.md)
* [线程安全](./Docs/MultiThread/线程安全.md)
* [ThreadLocal](./Docs/MultiThread/ThreadLocal.md)
* [ReentrantLock](./Docs/MultiThread/ReentrantLock.md)
* [同步相关知识](./Docs/MultiThread/同步相关知识.md)

### 网络
* [HTTPS](./Docs/Network/HTTPS.md)

### 设计模式
* [代理](./Docs/DesignPattern/代理.md)

### 算法
* [二叉树相关](./Docs/Algorithm/二叉树相关.md)
