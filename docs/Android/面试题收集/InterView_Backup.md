## KnowledgeSummary目录：

>此文件为备份文件，后续会从以下记录中问题以及知识点来分门别类的来总结。

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
  * Java基础知识总结
  * Java中的集合框架
  * 从源码角度分析ArrayList和Vector的区别
  * 从源码角度分析ArrayList和LinkedList的区别
  * 从源码角度分析HashMap的原理与实现
  * ConcurrentHashMap
  * LinkedHashMap
  * TreeMap

* Java进阶
  * Java中的注解
  * Java中的ClassLoader机制
  * JVM 
  * GC回收策略
  * GC算法
  * 类加载过程
  * 垃圾收集机制 对象创建，新生代与老年代
  * 可否用try-catch捕获OOM以避免其发生
  * 内存泄漏场景,如何定位及修复内存泄漏
  * 垃圾回收机制与调用System.gc()区别
  * NIO
  * Java四种引用
  * JavaPoet的使用指南




### Android
* UI
    * SurfaceView VS TextureView
    * ConstraintLayout
    * 常用Drawable
    * Drawable VS Bitmap
    * CoordinateLayout的原理分析

* Activity和Fragment
  * Activity的生命周期和启动模式
  * 简述Activity启动全部过程
  * launchMode
  * Activity调用方式
  * AlertDialog，Toast对Activity生命周期的影响
  * Dialog,PopupWindow,Activity区别
  * Activity与Fragment之间生命周期比较
  * 多层Fragment嵌套的时候setUserHint  
  * 下拉状态栏是不是影响activity的生命周期，如果在onStop的时候做了网络请求，onResume的时候怎么恢复  
  * Activity之间的通信方式
  * Android里面为什么要设计出Bundle而不是直接用Map结构
  * fragment 各种情况下的生命周期
  * ViewPager使用细节，如何设置成每次只初始化当前的Fragment，其他的不初始化
  * fragment之间传递数据的方式

* Service
  * Service的生命周期
  * 怎么启动service，service和activity怎么进行数据交互
  * [如何在后台下载任务, 并在通知栏显示进度](https://juejin.im/post/586072c861ff4b005820901d)

* BroadcastReceiver
  * 广播（动态注册和静态注册区别，有序广播和标准广播）
  * BroadcastReceiver，LocalBroadcastReceiver区别
  * 广播的使用场景

* ContentProvider
  * Android系统为什么会设计ContentProvider，进程共享和线程安全问题

* View
  * WebView优化（包括加载加速）
  * View事件传递
  * 封装view的时候怎么知道view的大小
  * 如何计算一个View的层级
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

### 开源库

* [Fresco]						
* [Tinker]					
* [ActivityRouter]
* [ARouter]				
* ButterKnife
* [EventBus]					
* [RxJava]					
  * RxJava
  * RxJava的功能与原理实现
  * RxJava简介及其源码解读？
  * RxJava的作用，优缺点
  * RxJava变换操作符map,flatMap,concatMap,buffer
* Retrofit
* [OKHTTP]	
* [LeakCanary]
* [Atlas]					
* [BlockCanary]
* Glide		
	* glide 使用什么缓存
	* Glide 内存缓存如何控制大小
* UETool

### 性能优化

* [绘制优化]
* [启动优化]
* [内存优化]
* [稳定性优化]
* [ANR分析]
* [耗电优化]
* [安装包优化]

(待整理)
* OOM定位及解决方案
* 安全、加固
* 如何保持应用的稳定性
* 性能优化如何分析systrace


### 图片相关

* [LruCache]
* [DiskLruCache]
* 图片加载原理
* 图片裁剪、压缩、旋转、滤镜
* 如何做图片缓存
* 如何防止加载大图OOM
* [图像显示原理]
* [Bitmap和Drawable	]
* [图片加载优化]
* Bitmap 使用时候注意什么
* bitmap recycler 相关
* 图片加载库相关，bitmap如何处理大图，如一张30M的大图，如何预防OOM

### Android其他

* [Art和Dalvik区别]
* [详解注解处理器APT技术]
* gradle
* [APK打包及安装过程]
* ActicityThread相关
* [IntentFilter匹配规则]
* [插件化]
* [组件化]
* [热修复方案对比]
* 埋点框架
* 画出Android的大体架构图
* 描述清点击AndroidStudio的build按钮后发生了什么
* 动态权限适配方案，权限组的概念
* 进程保活
* Android进程分类
* 是否熟悉Android jni开发，jni如何调用java层代码
* 多线程断点续传原理
* Appliction启动过程（App启动过程）
* App启动流程，从点击桌面开始 [link](http://www.androidos.net.cn/doc/day/2018-02-18/15384.md)
* 为什么不能在子线程更新UI
* App启动崩溃异常捕捉
* [数据库如何进行升级,SQLite增删改查的基础sql语句]
* App中唤醒其他进程的实现方式
* AndroidManifest的作用与理解
* Android中开启摄像头的主要步骤
* Application 和 Activity 的 context 对象的区别
* 差值器&估值器
* Android中进程内存的分配，能不能自己分配定额内存
* 视频加密传输
* 数据怎么压缩，数据的安全
* 插桩（打点等会用到）
* 模块化实现（好处，原因）
* 统计启动时长,标准
* 动态布局
* App是如何沙箱化，为什么要这么做；
* 权限管理系统(底层的权限是如何进行 grant 的)

### 多线程相关

* 死锁
* Android多进程
* Android进程间通信--Binder
* 多线程间通信
* 线程安全
* Java线程池
* 多线程池的优化（OKHTTP里面有）
* ThreadLocal
* ReentrantLock
* 同步相关知识
* 线程间 操作 List（多线程）
* synchronized与Lock的区别
* volatile
* JVM 内存区域 开线程影响哪块内存
* AIDL
* 进程与线程
* 并发集合了解哪些
* CAS介绍
* 开启线程的三种方式,run()和start()方法区别
* 多线程（关于AsyncTask缺陷引发的思考）
* 手写生产者/消费者模式
* 多进程场景遇见过么？
* 如何保证多线程读写文件的安全？
* volatile的原理
* synchronize的原理
* lock原理
* 线程如何关闭，以及如何防止线程的内存泄漏
* wait/notify
* 多线程：怎么用、有什么问题要注意；Android线程有没有上限，然后提到线程池的上限
* 线程间操作List

### 网络
* TCP/UDP的区别
* 数字证书包含的内容
* HTTPS
* Https请求慢的解决办法，DNS，携带数据，直接访问IP
* 网络请求缓存处理，okhttp如何处理网络缓存的
* https相关，如何验证证书的合法性，https中哪里用了对称加密，哪里用了非对称加密，对加密算法（如RSA）等是否有了解
* TCP与UDP区别与应用（三次握手和四次挥手）涉及到部分细节（如client如何确定自己发送的消息被server收到） HTTP相关 
* 提到过Websocket 问了WebSocket相关以及与socket的区别
* https握手过程，如何实现数据加密？客户端如何保证安全实现双重证书校验？请你设计一个登录功能，需要注意哪些安全问题


### 设计模式
* 代理
* 责任链（OKHTTP、Android事件分发）
* 观察者（RxJava）
* 装饰者（Java I/O Stream）
* 生产者消费者
* 单例
* 适配器模式，装饰者模式，外观模式的异同？


### 算法
* 排序，快速排序，堆排序的实现
* 常用数据结构简介
* 树：B+树的介绍,B+树
* 图：有向无环图的解释
* 阻塞队列BlockQueue
* 二叉树 深度遍历与广度遍历
* 二叉树，给出根节点和目标节点，找出从根节点到目标节点的路径
* 判断环（猜测应该是链表环）
* 链表反转
* 两个栈组成一个队列
* 一个无序，不重复数组，输出N个元素，使得N个元素的和相加为M，给出时间复杂度、空间复杂度。手写算法
* 合并多个单有序链表（假设都是递增的）
* 上一问扩展，海量数据，内存中放不下，怎么求出。
* String to Integer


### 操作系统
* 进程调度
* 进程状态
* 进程间通信的方式
* 逻辑地址与物理地址，为什么使用逻辑地址
* 为什么要有线程，而不是仅仅用进程
