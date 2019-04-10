本篇总结的知识点分类：

* [Java](#Java)
* [Android](#Android)
* [开源库](#开源库)
* [性能优化](#性能优化)
* [图片相关](#图片相关)
* [Android其他](#Android其他)
* [多线程相关](#多线程相关)
* [网络](#网络)
* [设计模式](#设计模式)
* [数据结构](#数据结构)
* [算法](#算法)
* [操作系统](#操作系统)
* [优秀网站](#优秀网站)


### Java
* Java基础知识
  * [基础知识](./Docs/Java/基础知识.md)
  * [Java内部类详解](./Docs/Java/Java内部类详解.md)


* 集合
  * [Java集合框架之Collections接口及实现类](./Docs/Java/Java集合框架之Collections接口及实现类.md)
  * LinkedHashMap					[link](./Docs/DataStructrue/LinkedHashMap.md)
  * SparseArray					[link](./Docs/DataStructrue/SparseArray.md)
  * TreeMap (支持红黑树排序)
  * ConcurrentHashMap				[link](./Docs/DataStructrue/ConcurrentHashMap.md)
  * HashTable
  * HashMap源码
      * HashMap实现原理和如何解决散列碰撞，HashMap底层为什么是线程不安全的
      * java8对hashmap的优化
      * hashmap和hashset区别，hash怎么散列的
      * HashMap的rehash扩容是怎么操作的


* Java进阶
  * java注解
  * JVM
  * 谈谈ClassLoader
  * GC回收策略
  * GC算法
  * 类加载过程
  * 依赖注入和控制反转				[link](./Docs/Java/依赖注入和控制反转.md)
  * 垃圾收集机制 对象创建，新生代与老年代
  * [可否用try-catch捕获OOM以避免其发生](./Docs/Java/可否用try-catch捕获OOM以避免其发生.md)
  * 内存泄漏场景
  * 内存泄露如何产生？
  * 如何定位及修复内存泄漏
  * 垃圾回收机制与调用System.gc()区别
  * [Java四种引用](./Docs/Java//Java四种引用.md)
  * 强引用置为null，会不会被回收？
  * NIO
  * [JavaPoet的使用指南](./Docs/Java//JavaPoet的使用指南.md)




### Android
* UI
    * SurfaceView VS TextureView	[link](./Docs/Android/UI/UI.md)
    * ConstraintLayout				[link](./Docs/Android/UI/UI.md)
    * 常用Drawable
    * Drawable VS Bitmap
    * CoordinateLayout.Behavior

* Activity和Fragment
  * Activity栈
  * 简述Activity启动全部过程
  * launchMode					[link](./Docs/Android/其他/launchMode.md)
  * Activity调用方式(显式&隐式)		[link](./Docs/Android/其他/Activity调用方式.md)
  * Activity 上有 Dialog 的时候按 home 键时的生命周期
  * 横竖屏切换的时候，Activity 各种情况下的生命周期
  * Dialog,PopupWindow,Activity区别
  * Activity与Fragment之间生命周期比较
  * 前台切换到后台，然后再回到前台，Activity生命周期回调方法。弹出Dialog，生命值周期回调方法
  * 多层Fragment嵌套的时候setUserHint  
  * 下拉状态栏是不是影响activity的生命周期，如果在onStop的时候做了网络请求，onResume的时候怎么恢复  
  * Activity之间的通信方式
    * bundle
    * 为什么使用Bundle不用HashMap传输数据 [Android里面为什么要设计出Bundle而不是直接用Map结构](https://github.com/android-cn/android-discuss/issues/142)
  * fragment 各种情况下的生命周期
  * ViewPager使用细节，如何设置成每次只初始化当前的Fragment，其他的不初始化
  * fragment之间传递数据的方式

* Service
  * service生命周期
  * Service两种启动方式			(startService和bindService)
  * 怎么启动service，service和activity怎么进行数据交互
    * 如何在后台下载任务, 并在Activity显示进度
  * Service的开启方式  

* BroadcastReceiver
  * 广播（动态注册和静态注册区别，有序广播和标准广播）
  * BroadcastReceiver，LocalBroadcastReceiver区别
  * 广播的使用场景

* ContentProvider
  * Android系统为什么会设计ContentProvider，进程共享和线程安全问题

* View
  * WebView优化（包括加载加速）
  * View事件传递   [link](./Docs/Android/其他//Android中的事件传递.md)
  * 封装view的时候怎么知道view的大小
  * 计算一个view的嵌套层级
  * 微信上消息小红点的原理
  * RecycleView的使用，原理，RecycleView优化 
  * ListView的优化 
  * LinearLayout、RelativeLayout、FrameLayout的特性、使用场景
  * view渲染  
  * ListView重用的是什么
  * 自定义View的属性引用attr，styleable里定义的名称可否与系统已经存在的name重复？当然是不可以的，编译器会预先检查系统已经存在或者之前已经定义重复的
  * Android为什么引入Parcelable
  * 有没有尝试简化Parcelable的使用
  * 序列化的作用，以及 Android 两种序列化的区别
  * [TextView性能瓶颈，渲染优化，以及StaticLayout的一些用处](https://www.jianshu.com/p/9f7f9213bff8)


*  Handler
   *  Handler如何在handleMessage方法拦截之前发出的message
   *  HandlerThread的原理
   *  IntentService的实现原理
   *  Handler实现机制（很多细节需要关注：如线程如何建立和退出消息循环等等
   *  Handler发消息给子线程，looper怎么启动
   *  Handler postDelay这个延迟是怎么实现的



### 开源库

* Fresco						[link](./Docs/Android/开源库/Fresco.md)
* Tinker						[link](./Docs/Android/开源库/tinker.md)
* ActivityRouter				[link](./Docs/Android/开源库/ActivityRouter.md)
* ARouter						[link](./Docs/Android/开源库/ARouter.md)
* ButterKnife
* EventBus						[link](./Docs/Android/开源库/EventBus.md)
* RxJava						[link](./Docs/Android/开源库/RxJava.md)
  *  RxJava
  * RxJava的功能与原理实现
  * RxJava简介及其源码解读？
  * RxJava的作用，优缺点
  * RxJava变换操作符map,flatMap,concatMap,buffer
* Retrofit						[link](./Docs/Android/开源库/Retrofit.md)
* OKHTTP						[link](./Docs/Android/开源库/OKHTTP.md)
* LeakCanary					[link](./Docs/Android/开源库/LeakCanary.md)
* Atlas							[link](./Docs/Android/开源库/atlast.md)
* BlockCanary
* Glide		
	* glide 使用什么缓存
	* Glide 内存缓存如何控制大小
* UETool

用到的一些开源框架，介绍一个看过源码的，内部实现过程。

### 性能优化

* 总览篇							[link](./Docs/Android/APP性能优化/总览篇.md)
* 性能优化1/5 -- 绘制优化			[link](./Docs/Android/APP性能优化/绘制优化.md)
	* Android系统显示原理
	* 布局优化					   
	* 避免过度绘制
	* 启动优化 						[link](./Docs/Android/APP性能优化/启动优化.md)
	* 合理刷新机制
	* 提升动画性能
	* 滑动卡顿
	* 工具篇	
* 性能优化2/5 -- 内存优化			[link](./Docs/Android/APP性能优化/内存优化.md)
	* 内存占用分析工具
	* 内存泄漏分析工具(LeakCanary)	
* 性能优化3/5 -- 稳定性优化		[link](./Docs/Android/APP性能优化/稳定性优化.md)
	* Java层Crash监控
	* Native层Crash监控
	* [ANR分析](./Docs/Android/APP性能优化/ANR分析.md)
* 性能优化4/5 -- 耗电优化			[link](./Docs/Android/APP性能优化/耗电优化.md)
* 性能优化5/5 -- 安装包优化		[link](./Docs/Android/APP性能优化/安装包优化.md)
* OOM定位及解决方案
* 安全、加固
* 如何保持应用的稳定性

* 性能优化如何分析systrace？


### 图片相关

* LRUCache						[link](./Docs/Android/图片相关/LruCache.md)
* DiskLruCache					[link](./Docs/Android/图片相关/DiskLruCache.md)
* 图片加载原理
* 图片裁剪、压缩、旋转、滤镜
* 如何做图片缓存
* 如何防止加载大图OOM
* WebP
* 图像显示原理					[link](./Docs/Android/图片相关/图像显示原理.md)
* Bitmap vs Drawable			[link](./Docs/Android/图片相关/bitmap_vs_drawable.md)
* 图片加载优化					[link](./Docs/Android/图片相关/图片加载优化.md)
* Bitmap 使用时候注意什么
* bitmap recycler 相关
* 图片加载库相关，bitmap如何处理大图，如一张30M的大图，如何预防OOM

### Android其他

* [Art和Dalvik区别](./Docs/Android/其他/Art和Dalvik区别.md)
* 详解注解处理器APT技术			[link](./Docs/Android/其他/详解APT.md)
* gradle
* 四大组件
* ANR 如何产生
* ANR 怎么分析
* APK打包及安装过程				[link](./Docs/Android/其他/APK打包及安装过程.md)
* ActicityThread相关
* 应用安装过程
* IntentFilter匹配规则			[link](./Docs/Android/其他/IntentFilter匹配规则.md)
* 插件化							[link](./Docs/Android/其他/插件化.md)
* 组件化							[link](./Docs/Android/其他/组件化.md)
* 热修复方案对比					[link](./Docs/Android/其他/热修复.md)
* 埋点框架
* 画出 Android 的大体架构图
* 描述清点击 Android Studio 的 build 按钮后发生了什么
* 动态权限适配方案，权限组的概念
* 进程保活
* Android进程分类
* 是否熟悉Android jni开发，jni如何调用java层代码
* 多线程断点续传原理
* Appliction启动过程（App启动过程）
  * App启动流程，从点击桌面开始
* 为什么不能在子线程更新UI
* App启动崩溃异常捕捉
* [数据库如何进行升级,SQLite增删改查的基础sql语句](./Docs/Android/其他/数据库如何进行升级,SQLite增删改查的基础sql语句.md)
* App中唤醒其他进程的实现方式
* AndroidManifest的作用与理解
* Android中开启摄像头的主要步骤
* Application 和 Activity 的 context 对象的区别
* 差值器&估值器
* Android中进程内存的分配，能不能自己分配定额内存
* 视频加密传输
* 数据怎么压缩，数据的安全
* MVP模式
* Android中为什么主线程不会因为Looper.loop()里的死循环卡死？
  * [主线程的工作原理](https://haldir65.github.io/2016/10/12/2016-10-12-How-the-mainThread-work/)
  > 这篇文章笔记有新意的思考点是：在2.2版本以前，这套机制是用我们熟悉的线程的wait和notify 来实现的,之前的Android版本用的是Java的线程wait和notify。

  >Android应用程序的主线程在进入消息循环过程前，会在内部创建一个Linux管道（Pipe），这个管道的作用是使得Android应用程序主线程在消息队列为空时可以进入空闲等待状态，并且使得当应用程序的消息队列有消息需要处理时唤醒应用程序的主线程。


### 多线程相关

* 死锁							[link](./Docs/MultiThread/死锁.md)
* AIDL	
* Android多进程					[link](./Docs/MultiThread/Android多进程.md)
* Android进程间通信--Binder		[link](./Docs/MultiThread/Android进程间通信--Binder.md)
* 多线程间通信					[link](./Docs/MultiThread/多线程间通信.md)
* 线程安全						[link](./Docs/MultiThread/线程安全.md)
* Java线程池
* 多线程池的优化（OKHTTP里面有）
* ThreadLocal					[link](./Docs/MultiThread/ThreadLocal.md)
* ReentrantLock					[link](./Docs/MultiThread/ReentrantLock.md)
* 同步相关知识					[link](./Docs/MultiThread/同步相关知识.md)
* 线程间 操作 List（多线程）
* synchronized与Lock的区别
* volatile
* JVM 内存区域 开线程影响哪块内存
* 进程与线程
* 并发集合了解哪些
* CAS介绍
* 开启线程的三种方式,run()和start()方法区别
* 多线程（关于AsyncTask缺陷引发的思考）
* 手写生产者/消费者模式
* 多进程场景遇见过么？
* concurrenthashmap
* 如何保证多线程读写文件的安全？
* volatile的原理
* synchronize的原理
* lock原理
* 线程如何关闭，以及如何防止线程的内存泄漏
* wait/notify
* 多线程：怎么用、有什么问题要注意；Android线程有没有上限，然后提到线程池的上限
* 线程死锁的4个条件
* 线程间操作List

### 网络
* TCP/UDP的区别
* 数字证书包含的内容
* HTTPS							[link](./Docs/Network/HTTPS.md)
* Https请求慢的解决办法，DNS，携带数据，直接访问IP
* 网络请求缓存处理，okhttp如何处理网络缓存的
* https相关，如何验证证书的合法性，https中哪里用了对称加密，哪里用了非对称加密，对加密算法（如RSA）等是否有了解
* TCP与UDP区别与应用（三次握手和四次挥手）涉及到部分细节（如client如何确定自己发送的消息被server收到） HTTP相关 
* 提到过Websocket 问了WebSocket相关以及与socket的区别
* https握手过程，如何实现数据加密？客户端如何保证安全实现双重证书校验？请你设计一个登录功能，需要注意哪些安全问题


### 设计模式
* 代理，动态代理（Retrofit）					[link](./Docs/DesignPattern/代理.md)
* 责任链（OKHTTP、Android事件分发）
* 观察者（RxJava）
* 装饰者（Java I/O Stream）
* 生产者消费者
* 单例
* 适配器模式，装饰者模式，外观模式的异同？

### 数据结构
* 排序，快速排序，堆排序的实现
* 树：B+树的介绍
* 图：有向无环图的解释
* 阻塞队列BlockQueue
* 集合 Set实现 Hash 怎么防止碰撞
* 二叉树 深度遍历与广度遍历
* B树、B+树
* 常用数据结构简介
* 判断环（猜测应该是链表环）
* 链表反转
* arraylist和linkedlist的区别，以及应用场景
* 二叉树，给出根节点和目标节点，找出从根节点到目标节点的路径
* HashMap的实现，与HashSet的区别
* 翻转一个单项链表
* HashSet与HashMap怎么判断集合元素重复


### 算法
* 快速排序
* 两个栈组成一个队列
* [二叉树相关]					(./Docs/Algorithm/二叉树相关.md)
* 一个无序，不重复数组，输出N个元素，使得N个元素的和相加为M，给出时间复杂度、空间复杂度。手写算法
* 合并多个单有序链表（假设都是递增的）
* 上一问扩展，海量数据，内存中放不下，怎么求出。
* string to integer


### 操作系统
* 进程调度
* 进程状态
* 进程间通信的方式
* 逻辑地址与物理地址，为什么使用逻辑地址
* 为什么要有线程，而不是仅仅用进程？


### 优秀网站					[link](./Docs/Android/优秀网站.md)

* 插桩（打点等会用到）
* 模块化实现（好处，原因）
* 统计启动时长,标准
* 动态布局
* App 是如何沙箱化，为什么要这么做；
* 权限管理系统（底层的权限是如何进行 grant 的）