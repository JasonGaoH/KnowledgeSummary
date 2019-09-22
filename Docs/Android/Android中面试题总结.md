# 前言
自己一直做Java、Android相关的知识总结->[KnowledgeSummary系列](https://github.com/JasonGaoH/KnowledgeSummary)。这个GitHub的repo想作为自己对于Android核心知识点以及一些面试题的总结，因为很多知识点理解的不够深刻，所以通过这个来强迫自己做总结，目前已经初具规模，我基本每天都会更新这个repo，后续还会持续更新下去，大家有兴趣可以点个star关注下，感谢。

在做这个知识总结系列，发现有的比较零散的面试题不太适合开个doc文档来写，于是想着把这些零散的面试题做个总结，所以想收录在这篇文章里面，并且每个面试题尽量提供一个参考答案，因为如果只是把面试题列出来，我举得一点意思都没有，这样的面试题网上一搜一大把，所以我希望为每一个面试题提供一个参考，但不能保证这些答案完全正确。

我的目标是这份面试题能够长期更新，因为比较多，暂时就不分类了。

[原文链接](https://github.com/JasonGaoH/KnowledgeSummary/blob/master/Docs/Android/Android%E4%B8%AD%E9%9D%A2%E8%AF%95%E9%A2%98%E6%80%BB%E7%BB%93.md)

[如何计算一个View的层级](#如何计算一个view的层级)
[Art和Dalvik区别](#art和dalvik区别)
[socket判断http请求或http响应的传输结束](#socket判断http请求或http响应的传输结束)
[OnLowMemory和OnTrimMemory的比较](#onlowmemory和ontrimmemory的比较)
[BlockCanary核心原理分析](#blockcanary核心原理分析)
[onRestart 什么时候调用](#onrestart-什么时候调用)
[Handler 如何防止内存泄漏](#handler-如何防止内存泄漏)
[TCP可靠性的保证机制总结](#tcp可靠性的保证机制总结)
[synchronized 和 ReentrantLock 的区别和使用选择](#synchronized-和-reentrantlock-的区别和使用选择)
[synchronized，volitale的异同](#synchronizedvolitale的异同)
[SparseArray和ArrayMap使用场景](#sparsearray和arraymap使用场景)
[Android 中 getRawX()和 getX()区别](#android-中-getrawx和-getx区别)
[Android中的MotionEvent手势事件](#android中的motionevent手势事件)
[onInterceptTouchEvent()函数与 onTouchEvent()的区别](#onintercepttouchevent函数与-ontouchevent的区别)
[Http 1.0和Http 2.0的区别](#http-10和http-20的区别)
[View.getLocationInWindow和View.getLocationOnScreen区别](#viewgetlocationinwindow和viewgetlocationonscreen区别)
[http和https的区别](#http和https的区别)
[为什么要有内部类,静态内部类和普通内部类区别是什么](#为什么要有内部类静态内部类和普通内部类区别是什么)
[Java中四种线程池的总结](#java中四种线程池的总结)
[Intent传递数据的限制大小](#intent传递数据的限制大小)
[onStartCommand 的几种模式](#onstartcommand-的几种模式)
[RelativeLayout的onMeasure方法是怎么Measure的](#relativelayout的onmeasure方法是怎么measure的)
[主线程的死循环是否一致耗费CPU资源](#主线程的死循环是否一致耗费cpu资源)
[Service 和 IntentService 的区别](#service-和-intentservice-的区别)
[SQLite增删改查以及升级的sql语句](#sqlite增删改查以及升级的sql语句)
[Service的生命周期](#service的生命周期)[Activity之间的通信方式](#activity之间的通信方式)
[SurfaceView和TextureView的区别](#surfaceview和textureview的区别)
[onSaveInstanceState和onRestoreInstanceState调用时机](#onsaveinstancestate和onrestoreinstancestate调用时机)
[LeakCanary原理](#leakcanary原理)
[Android系统为什么会设计ContentProvider](#android系统为什么会设计contentprovider)
[Service和Activity通信](#service和activity通信)
[OOM 是否可以try catch](#oom-是否可以try-catch)
[AlertDialog，Toast 对Activity生命周期的影响](#alertdialogtoast-对activity生命周期的影响)
[HTTP与TCP的区别和联系](#http与tcp的区别和联系)
[viewstub可以多次inflate么?多次inflate会怎样?](#viewstub可以多次inflate么多次inflate会怎样)
[onWindowFocusChanged 执行时机](#onwindowfocuschanged-执行时机)

## 如何计算一个View的层级
``【参考】:``
```java
int i = 0;
private void getParents(ViewParent view){
    if (view.getParent() == null) { 
        Log.v("tag", "最终==="+i); return;
    }
    i++;
    ViewParent parent = view.getParent(); 
    Log.v("tag", "i===="+i);
    Log.v("tag", "parent===="+parent.toString());
    getParents(parent); 
}
```

> 因为``public abstract class ViewGroup extends View implements ViewParent``，
ViewGroup是ViewParent的实现类，所以可以直接转， LinearLayout是ViewGroup 的子类。

## Art和Dalvik区别
``【参考】:``
Android 4.4发布了一个ART运行时，准备用来替换掉之前一直使用的Dalvik虚拟机

Art: Android Runtime，编译机制：AOT，预编译机制
Dalvik：编译机制：JIT，即时编译

ART 的机制与 Dalvik 不同。在Dalvik下，应用每次运行的时候，字节码都需要通过即时编译器（JIT，Just-In-Time）转换为机器码，这会拖慢应用的运行效率，而在ART 环境中，`应用在安装的时候，字节码就会预先编译成机器码`，使其成为真正的本地应用。这个过程叫做预编译（AOT,Ahead-Of-Time）。这样的话，应用的启动(首次)和执行都会变得更加快速。

什么是Dalvik
> Dalvik是Google公司自己设计用于Android平台的Java虚拟机。Dalvik虚拟机是Google等厂商合作开发的Android移动设备平台的核心组成部分之一，它可以支持已转换为.dex(即Dalvik Executable)格式的Java应用程序的运行，.dex格式是专为Dalvik应用设计的一种压缩格式，适合内存和处理器速度有限的系统。Dalvik经过优化，允许在有限的内存中同时运行多个虚拟机的实例，并且每一个Dalvik应用作为独立的Linux进程执行。独立的进程可以防止在虚拟机崩溃的时候所有程序都被关闭。

什么是ART

> Android操作系统已经成熟，Google的Android团队开始将注意力转向一些底层组件，其中之一是负责应用程序运行的Dalvik运行时。Google开发者已经花了两年时间开发更快执行效率更高更省电的替代ART运行时。ART代表Android Runtime,其处理应用程序执行的方式完全不同于Dalvik，Dalvik是依靠一个Just-In-Time(JIT)编译器去解释字节码。开发者编译后的应用代码需要通过一个解释器在用户的设备上运行，这一机制并不高效，但让应用能更容易在不同硬件和架构上运行。ART则完全改变了这套做法，在应用安装的时候就预编译字节码到机器语言，这一机制叫Ahead-Of-Time(AOT)编译。在移除解释代码这一过程后，应用程序执行将更有效率，启动更快。

ART优点：
* 系统性能的显著提升
* 应用启动更快、运行更快、体验更流畅、触感反馈更及时
* 更长的电池续航能力
* 支持更低的硬件

ART缺点：
* 更大的存储空间占用，可能会增加10%-20%
* 更长的应用安装时间

## socket判断http请求或http响应的传输结束

``【参考】:``先把 header 直到\r\n\r\n 整个地址记录下来
> 如果是短连接，没有启用 keepalive，则可以通过是否关闭了连接来判断是否传输结束， 即在读取时可判断 read() != -1。传输完毕就关闭 connection，即 recv 收到 0 个字节。
> 如果是长连接，那么一个 socket(tcp)可能发送和接收多次请求，那么如何判断每次的 响应已经接收?
> 1. 先读请求头，一直到\r\n\r\n 说明请求头结束，然后解析 http 头，如果 Content-Length=x 存在，则知道 http 响应的长度为 x。从头的末尾直接读取 x 字节就是响应内容。
> 2. 如果 Content-Length=x 不存在，那么头类型为 Transfer-Encoding: chunked 说明响应 的长度不固定，则在响应头结束后标记第一段流的长度，即直到流里有\r\n0\r\n\r\n 结束
> 3. 如果 recv 返回 SOCKET_ERROR 时，说明对方已经断开连接，但是可能是非正常断开 (断网或者客户端进程结束

## OnLowMemory和OnTrimMemory的比较
``【参考】: ``OnLowMemory 被回调时，已经没有后台进程;而 onTrimMemory 被回调时，还有后台进程。
OnLowMemory 是在最后一个后台进程被杀时调用，一般情况是 low memory killer 杀进程 后触发;而 OnTrimMemory 的触发更频繁，每次计算进程优先级时，只要满足条件，都会触发。
通过一键清理后，OnLowMemory 不会被触发，而 OnTrimMemory 会被触发一次。

## BlockCanary核心原理分析
``【参考】: ``
```java
public static void loop() {
        final Looper me = myLooper();
        if (me == null) {
            throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
        }
        final MessageQueue queue = me.mQueue;

        // Make sure the identity of this thread is that of the local process,
        // and keep track of what that identity token actually is.
        Binder.clearCallingIdentity();
        final long ident = Binder.clearCallingIdentity();

        for (;;) {
            Message msg = queue.next(); // might block
            if (msg == null) {
                // No message indicates that the message queue is quitting.
                return;
            }

            // This must be in a local variable, in case a UI event sets the logger
            final Printer logging = me.mLogging;
            if (logging != null) {
                logging.println(">>>>> Dispatching to " + msg.target + " " +
                        msg.callback + ": " + msg.what);
            }

            final long slowDispatchThresholdMs = me.mSlowDispatchThresholdMs;

            final long traceTag = me.mTraceTag;
            if (traceTag != 0 && Trace.isTagEnabled(traceTag)) {
                Trace.traceBegin(traceTag, msg.target.getTraceName(msg));
            }
            final long start = (slowDispatchThresholdMs == 0) ? 0 : SystemClock.uptimeMillis();
            final long end;
            try {
                msg.target.dispatchMessage(msg);
                end = (slowDispatchThresholdMs == 0) ? 0 : SystemClock.uptimeMillis();
            } finally {
                if (traceTag != 0) {
                    Trace.traceEnd(traceTag);
                }
            }
            if (slowDispatchThresholdMs > 0) {
                final long time = end - start;
                if (time > slowDispatchThresholdMs) {
                    Slog.w(TAG, "Dispatch took " + time + "ms on "
                            + Thread.currentThread().getName() + ", h=" +
                            msg.target + " cb=" + msg.callback + " msg=" + msg.what);
                }
            }

            if (logging != null) {
                logging.println("<<<<< Finished to " + msg.target + " " + msg.callback);
            }

            // Make sure that during the course of dispatching the
            // identity of the thread wasn't corrupted.
            final long newIdent = Binder.clearCallingIdentity();
            if (ident != newIdent) {
                Log.wtf(TAG, "Thread identity changed from 0x"
                        + Long.toHexString(ident) + " to 0x"
                        + Long.toHexString(newIdent) + " while dispatching to "
                        + msg.target.getClass().getName() + " "
                        + msg.callback + " what=" + msg.what);
            }

            msg.recycleUnchecked();
        }
    }
```

原理:我们只需要计算打印这两天 log 的时间差，就能得到 dispatchMessage 的耗时，android 提供了 Looper.getMainLooper().setMessageLogging(Printer printer)来设置这个 logging 对 象，所以只要自定义一个 Printer，然后重写 println(String x)方法即可实现耗时统计了。
通过在 DispatchMessage 的方法的执行时间，来判断卡顿，管是哪种回调方式，回调一定发 生在 UI 线程。因此如果应用发生卡顿，一定是在 dispatchMessage 中执行了耗时操作。我 们通过给主线程的 Looper 设置一个 Printer，打点统计 dispatchMessage 方法执行的时间， 如果超出阀值，表示发生卡顿，则 dump 出各种信息，提供开发者分析性能瓶颈。 mBlockListener.onBlockEvent。

## onRestart 什么时候调用
``【参考】: ``
* 1、按下 home 键执行，再次打开这个 demo 执行;onRestart()--->onStart()--->onResume() 三个方法。
* 2、点击界面的 btn，跳转到另一个 Activity1，从 Activity1 返回，会执行如下图 2;onRestart()- -->onStart()--->onResume()三个方法
* 3、切换到其他的应用，从其他应用切换回来。onRestart()--->onStart()--->onResume()三个 方法

## Handler 如何防止内存泄漏
``【参考】: ``
除了写弱引用这个方法后，还有一个就是 handler.removeCallbacksAndMessages(null);，就 是移除所有的消息和回调，简单一句话就是清空了消息队列。注意，不要以为你 post 的是 个 Runnable 或者只是 sendEmptyMessage。你可以看一下源码，在 handler 里面都是会把 这些转成正统的 Message，放入消息队列里面，所以清空队列就意味着这个 Handler 直接被 打成原型了，当然也就可以回收了。

## TCP可靠性的保证机制总结
``【参考】: ``
1. 检验和
2. 序列号 
3. 确认应答机制(ACK)
4. 超时重传机制
5. 连接管理机制 
6. 流量控制 
7. 拥塞控制

## synchronized 和 ReentrantLock 的区别和使用选择
``【参考】: ``
1. 使用 synchronized 获得的锁存在一定缺陷
> * 不能中断一个正在试图获得锁的线程
> * 试图获得锁时不能像 ReentrantLock 中的 trylock 那样设定超时时间 ，当一个线程获得了对象锁后，其他线程访问这个同步方法时，必须等待或阻塞，如果那个线程发生了死循环， 对象锁就永远不会释放
> * 每个锁只有单一的条件，不像 condition 那样可以设置多个

2. 尽管 synchronized 存在上述的一些缺陷，在选择上还是以 synchronized 优先

> * 如果 synchronized 关键字适合程序，尽量使用它，可以减少代码出错的几率和代码数量; (减少出错几率是因为在执行完 synchronized 包含完的最后一句语句后，锁会自动释放， 不需要像 ReentrantLock 一样手动写 unlock 方法;)
> * 如果特别需要 Lock/Condition 结构提供的独有特性时，才使用他们;(比如设定一个线程 长时间不能获取锁时设定超时时间或自我中断等功能。)
> * 许多情况下可以使用 java.util.concurrent 包中的一种机制，它会为你处理所有的加锁情况; (比如当我们在多线程环境下使用 HashMap 时，可以使用 ConcurrentHashMap 来处理多线程并发)

## synchronized，volitale的异同
``【参考】: ``
1. volatile 本质是在告诉 jvm 当前变量在寄存器中的值是不确定的,需要从主存中读取,synchronized 则是锁定当前变量,只有当前线程可以访问该变量,其他线程被阻塞住
2. volatile 仅能使用在变量级别,synchronized 则可以使用在变量,方法
3. volatile 仅能实现变量的修改可见性,但不具备原子特性,而 synchronized 则可以保证变量 的修改可见性和原子性
4. volatile 不会造成线程的阻塞,而 synchronized 可能会造成线程的阻塞
6. volatile 标记的变量不会被编译器优化,而 synchronized标记的变量可以被编译器优化

## SparseArray和ArrayMap使用场景
``【参考】: ``
假设数据量都在千级以内的情况下:
1. 假设 key 的类型已经确定为 int 类型。那么使用 SparseArray，由于它避免了自己主动装箱的过程，假设key为long类型，它还提供了一个 LongSparseArray 来确保key为 long类型时的使用
2. 假设key类型为其他的类型，则使用ArrayMap
SparseArray比HashMap 更省内存，在某些条件下性能更好，主要是由于它避免了对 key 的自己主动装箱(int转为Integer 类型)，它内部则是通过两个数组来进行数据存储的。一个存储 key，另外一个存储value，为了优化性能，它内部对数据还採取了压缩的方式来表示 稀疏数组的数据，从而节约内存空间。我们从源代码中能够看到key和value各自是用数组

## Android 中 getRawX()和 getX()区别
* getRawX( )即表示的是点击的位置距离屏幕的坐标
* getX( )即表示的点击的位置相对于本身的坐标

## Android中的MotionEvent手势事件
``【参考】: ``
* MotionEvent.ACTION_DOWN:当屏幕检测到第一个触点按下之后就会触发到这个事件。
* MotionEvent.ACTION_MOVE:当触点在屏幕上移动时触发，触点在屏幕上停留也是会触发 的，主要是由于它的灵敏度很高，而我们的手指又不可能完全静止(即使我们感觉不到移动， 但其实我们的手指也在不停地抖动)。
* MotionEvent.ACTION_POINTER_DOWN:当屏幕上已经有触点处于按下的状态的时候，再有 新的触点被按下时触发。
* MotionEvent.ACTION_POINTER_UP:当屏幕上有多个点被按住，松开其中一个点时触发(即 非最后一个点被放开时)触发。
* MotionEvent.ACTION_UP:当最后一个触点松开时被触发。
* MotionEvent.ACTION_SCROLL:非触摸滚动，主要是由鼠标、滚轮、轨迹球触发。
* MotionEvent.ACTION_CANCEL:不是由用户直接触发，有系统再需要的时候触发，例如当父view通过使函数 onInterceptTouchEvent()返回 true,从子view拿回处理事件的控制权是，就 会给子 view 发一个ACTION_CANCEL事件，这里了 view 就再也不会收到事件了。可以将其视为ACTION_UP事件对待。

## onInterceptTouchEvent()函数与 onTouchEvent()的区别
``【参考】: ``
* onInterceptTouchEvent()是用于处理事件(类似于预处理，当然也可以不处理)并改变事 件的传递方向，也就是决定是否允许 Touch 事件继续向下(子 view)传递，一但返回 True (代表事件在当前的 viewGroup 中会被处理)，则向下传递之路被截断(所有子view将没有机会参与Touch 事件)，同时把事件传递给当前的view的onTouchEvent()处理;返回 false， 则把事件交给子view的onInterceptTouchEvent()
* onTouchEvent()用于处理事件，返回值决定当前view是否消费(consume)了这个事件， 也就是说在当前view在处理完Touch事件后，是否还允许Touch事件继续向上(父 view) 传递，一但返回 True，则父 view 不用操心自己来处理 Touch 事件。返回 true，则向上传递 给父 view(注:可能你会觉得是否消费了有关系吗，反正我已经针对事件编写了处理代码? 答案是有区别!比如 ACTION_MOVE 或者 ACTION_UP 发生的前提是一定曾经发生了 ACTION_DOWN，如果你没有消费 ACTION_DOWN，那么系统会认为 ACTION_DOWN 没有 发生过，所以 ACTION_MOVE 或者 ACTION_UP 就不能被捕获。)

## Http 1.0和Http 2.0的区别
``【参考】: ``
> HTTP1.0 
无状态、无连接
> HTTP1.1
持久连接
请求管道化
增加缓存处理(新的字段如 cache-control)
增加 Host 字段、支持断点传输等(把文件分成几部分)
> HTTP2.0 
二进制分帧
多路复用(或连接共享)
头部压缩

## View.getLocationInWindow和View.getLocationOnScreen区别
``【参考】: ``
* View.getLocationInWindow(int[] location)是获得控件在其父窗口中的坐标位置
* View.getLocationOnScreen(int[] location)是获得控件在其整个屏幕上的坐标位置

## http和https的区别
``【参考】: ``
https 通信过程
1. 在使用 HTTPS 是需要保证服务端配置正确了对应的安全证书
2. 客户端发送请求到服务端
3. 服务端返回公钥和证书到客户端
4. 客户端接收后会验证证书的安全性,如果通过则会随机生成一个随机数,用公钥对其加密, 发送到服务端
5. 服务端接受到这个加密后的随机数后会用私钥对其解密得到真正的随机数,随后用这个随机数当做私钥对需要发送的数据进行对称加密
6. 客户端在接收到加密后的数据使用私钥(即生成的随机值)对数据进行解密并且解析数据 呈现结果给客户
7. SSL 加密建立

HTTPS 和 HTTP 的区别主要如下:
1. https 协议需要到 ca 申请证书，一般免费证书较少，因而需要一定费用。
2. http 是超文本传输协议，信息是明文传输，https 则是具有安全性的 ssl 加密传输协议。 
3. http 和 https 使用的是完全不同的连接方式，用的端口也不一样，前者是 80，后者是 443。 
4. http 的连接很简单，是无状态的;HTTPS 协议是由 SSL+HTTP 协议构建的可进行加密传 输、身份认证的网络协议，比 http 协议安全。

## 为什么要有内部类,静态内部类和普通内部类区别是什么

1. 内部类一般只为其外部类使用;
2. 内部类提供了某种进入外部类的窗户; 
3. 也是最吸引人的原因，每个内部类都能独立地继承一个接口，而无论外部类是否已经继 承了某个接口。因此，内部类使多重继承的解决方案变得更加完整。

内部类和外部类区别
1. 内部类是一个编译时概念，编译后外部类及其内部类会生成两个独立的 class 文件: OuterClass.class 和 OuterClass$InnerClass.class 
2. 内部类可以直接访问外部类的元素，但是外部类不可以直接访问内部类的元素
3. 外部类可以通过内部类引用间接访问内部类元素
4. 对于静态内部类来说，静态内部类是不依赖于外部类的，也就说可以在不创建外部类对 象的情况下创建内部类的对象。另外，静态内部类是不持有指向外部类对象的引用的

静态内部类和普通内部类
静态内部类与非静态内部类之间存在一个最大的区别，我们知道非静态内部类在编译完成之 后会隐含地保存着一个引用，该引用是指向创建它的外围内。 但是静态内部类却没有。没有这个引用就意味着:
静态内部类的创建是不需要依赖于外围类，可以直接创建 静态内部类不可以使用任何外围类的非 static 成员变量和方法，而内部类则都可以
注意:
1. 非静态内部类中不允许定义静态成员 
2. 外部类的静态成员不可以直接使用非静态内部类 
3. 静态内部类，不能访问外部类的实例成员，只能访问外部类的类成

## Java中四种线程池的总结
``【参考】: ``具体的 4 种常用的线程池实现如下:(返回值都是 ExecutorService)

1. Executors.newCacheThreadPool()
可缓存线程池，先查看池中有没有以前建立的线程， 如果有，就直接使用。如果没有，就建一个新的线程加入池中，缓存型池子通常用于执行一 些生存期很短的异步型任务。
```java
ExecutorService cachedThreadPool = Executors.newCachedThreadPool();
```
2. Executors.newFixedThreadPool(int n)
创建一个定长线程池，以共享的无界队列方式来运行这些线程
```java
ExecutorService fixedThreadPool = Executors.newFixedThreadPool(3);
```
3. Executors.newScheduledThreadPool(int n):
创建一个定长线程池，支持定时及周期性任务执行
```java
ScheduledExecutorService scheduledThreadPool = Executors.newScheduledThreadPool(5);
```
4. Executors.newSingleThreadExecutor():创建一个单线程化的线程池，它只会用唯一的工作线程来执行任务，保证所有任务按照指定顺序(FIFO, LIFO, 优先级)执行。
```java
ExecutorService singleThreadExecutor = Executors.newSingleThreadExecutor();
```

```java
public ThreadPoolExecutor(int corePoolSize,
int maximumPoolSize,
long keepAliveTime,
TimeUnit unit, BlockingQueue<Runnable> workQueue) {
    this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue, Executors.defaultThreadFactory(), defaultHandler);
}
```
* corePoolSize:核心池大小，意思是当超过这个范围的时候，就需要将新的线程放到等待队列 中了即 workQueue;
* maximumPoolSize:线程池最大线程数量，表明线程池能创建的最大线程数 
* keepAlivertime:当活跃线程数大于核心线程数，空闲的多余线程最大存活时间。 
* unit:存活时间的单位
* workQueue:存放任务的队列---阻塞队列 
* handler:超出线程范围(maximumPoolSize)和队列容量的任务的处理程序

我们执行线程时都会调用到 ThreadPoolExecutor 的 execute()方法，现在我们来看看这个方法的源码(就是下面这段代码了，这里面有一些注释解析)，我直接来解释一下吧:在这段代码中我们至少要看懂一个逻辑:当当前线程数小于核心池线程数时，只需要添加一个线程并且启动它，如果线程数数目大于核心线程池数目，我们将任务放到 workQueue中，如果连WorkQueue满了，那么就要拒绝任务了。

## Intent传递数据的限制大小
``【参考】: ``
用Intent传递数据，实际上走的是跨进程通信（IPC），跨进程通信需要把数据从内核copy到进程中，每一个进程有一个接收内核数据的缓冲区，默认是1M；如果一次传递的数据超过限制，就会出现异常。

不同厂商表现不一样有可能是厂商修改了此限制的大小，也可能同样的对象在不同的机器上大小不一样。

## onStartCommand 的几种模式
``【参考】: ``
> START_NOT_STICKY
如果返回 START_NOT_STICKY，表示当 Service 运行的进程被 Android 系统强制杀掉之后， 不会重新创建该Service。当然如果在其被杀掉之后一段时间又调用了startService，那么该 Service 又将被实例化。那什么情境下返回该值比较恰当呢?
如果我们某个 Service 执行的工作被中断几次无关紧要或者对 Android 内存紧张的情况下需 要被杀掉且不会立即重新创建这种行为也可接受，那么我们便可将 onStartCommand 的返 回值设置为 START_NOT_STICKY。
举个例子，某个 Service 需要定时从服务器获取最新数据:通过一个定时器每隔指定的 N 分 钟让定时器启动 Service 去获取服务端的最新数据。当执行到 Service 的 onStartCommand 时，在该方法内再规划一个 N 分钟后的定时器用于再次启动该 Service 并开辟一个新的线程 去执行网络操作。假设 Service 在从服务器获取最新数据的过程中被 Android 系统强制杀掉， Service 不会再重新创建，这也没关系，因为再过 N 分钟定时器就会再次启动该 Service 并重 新获取数据。
> START_STICKY
如果返回 START_STICKY，表示 Service 运行的进程被 Android 系统强制杀掉之后，Android 系统会将该 Service 依然设置为 started 状态(即运行状态)，但是不再保存 onStartCommand 方法传入的 intent 对象，然后 Android 系统会尝试再次重新创建该 Service，并执行 onStartCommand 回调方法，但是 onStartCommand 回调方法的 Intent 参数为 null，也就是 onStartCommand 方法虽然会执行但是获取不到 intent 信息。如果你的 Service 可以在任意 时刻运行或结束都没什么问题，而且不需要 intent 信息，那么就可以在 onStartCommand 方 法中返回 START_STICKY，比如一个用来播放背景音乐功能的 Service 就适合返回该值。
> START_REDELIVER_INTENT
如果返回 START_REDELIVER_INTENT，表示 Service 运行的进程被 Android 系统强制杀掉之后，与返回START_STICKY的情况类似，Android 系统会将再次重新创建该 Service，并执行 onStartCommand 回调方法，但是不同的是，Android 系统会再次将 Service 在被杀掉之前 最后一次传入 onStartCommand 方法中的 Intent 再次保留下来并再次传入到重新创建后的 Service 的 onStartCommand 方法中，这样我们就能读取到 intent 参数。只要返回 START_REDELIVER_INTENT，那么 onStartCommand 重的 intent 一定不是 null。如果我们的 Service 需要依赖具体的 Intent 才能运行(需要从 Intent 中读取相关数据信息等)，并且在强 制销毁后有必要重新创建运行，那么这样的 Service 就适合返回 START_REDELIVER_INTENT

## RelativeLayout的onMeasure方法是怎么Measure的
``【参考】: `` 发现 RelativeLayout 会根据 2次排列的结果对子View各做一次 measure。 而在做横向的测量时，纵向的测量结果尚未完成，只好暂时使用 myHeight 传入子View系统。
这样必然会导致子View的高度和 RelativeLayout 的高度不同时，第 3 点中所说的优化会失效，在View系统足够复杂时，效率问题就会出现。

## 主线程的死循环是否一致耗费CPU资源
``【参考】: ``  在主线程使用 Looper.loop 可以保证主线程一直在运行，事实上，在 Looper.loop 死循环之 前，已经创建了一个 Binder 线程:
thread.attach 会建立 Binder 通道，创建新线程。attach(false)会创建一个 ApplicaitonThread 的 Binder 线程，用于接受 AMS 发来的消息，该 Binder 线程通过 ActivityThread 的 H 类型 Handler 将消息发送给主线程。ActivityThread 并不是线程类，只是它运行在主线程。
Handler 底层采用 Linux 的 pipe/epoll 机制，MessageQueue 没有消息的时候，便阻塞在 Looper.mQueue.next 方法中，此时主线程会释放 CPU 资源进入休眠，直到下个事件到达， 当有新消息的时候，通过往 pipe 管道写数据来唤醒主线程工作。所以主线程大多数时候处 于休眠状态，不会阻塞。

## Service 和 IntentService 的区别

``【参考】: ``
1. IntentService 是继承并处理异步请求的一个类，在 IntentService 内有一个工作 线程来处理耗时操作，启动 IntentService 的方式和启动传统的 Service 一样，同时，当任务 执行完后，IntentService 会自动停止，而不需要我们手动去控制或 stopSelf()。另外，可以启 动 IntentService 多次，而每一个耗时操作会以工作队列的方式在 IntentService 的 onHandleIntent 回调方法中执行，并且，每次只会执行一个工作线程，执行完第一个再执行 第二个，以此类推。
2. 子类需继承 IntentService 并且实现里面的 onHandlerIntent 抽象方法来处理 intent 类型 的任务请求。
3. 子类需要重写默认的构造方法，且在构造方法中调用父类带参数的构造方法。
4. IntentService 类内部利用 HandlerThread+Handler 构建了一个带有消息循环处理机制的 后台工作线程，客户端只需调用 Content#startService(Intent)将 Intent 任务请求放入后台工 作队列中，且客户端无需关注服务是否结束，非常适合一次性的后台任务。比如浏览器下载 文件，退出当前浏览器之后，下载任务依然存在后台，直到下载文件结束，服务自动销毁。 只要当前 IntentService 服务没有被销毁，客户端就可以同时投放多个 Intent 异步任务请求， IntentService 服务端这边是顺序执行当前后台工作队列中的 Intent 请求的，也就是每一时刻 只能执行一个 Intent 请求，直到该 Intent 处理结束才处理下一个 Intent。因为 IntentService 类内部利用 HandlerThread+Handler 构建的是一个单线程来处理异步任务。

## SQLite增删改查以及升级的sql语句
``【参考】: ``
``` java
public SQLiteOpenHelper(Context context, String name, CursorFactory factory, int version) {
        this(context, name, factory, version, null);
    }

    public SQLiteDatabase getWritableDatabase() {
        synchronized (this) {
            return getDatabaseLocked(true);
        }
    }

  private SQLiteDatabase getDatabaseLocked(boolean writable) {
      .......
      db.beginTransaction();
      try {
              if (version == 0) {
                   onCreate(db);
              } else {
                   if (version > mNewVersion) {
                         onDowngrade(db, version, mNewVersion);
                   } else {
                         onUpgrade(db, version, mNewVersion);
                   }
              }
               db.setVersion(mNewVersion);
                db.setTransactionSuccessful();
              } finally {
                 db.endTransaction();
              }
  }

```
onUpgrade()会被触发，可以在该方法中编写数据库升级逻辑。具体的数据库升级逻辑示例可参考这里.

常用的SQL增删改查：
- 增：INSERT INTO table_name (列1, 列2,…) VALUES (值1, 值2,….)
- 删： DELETE FROM 表名称 WHERE 列名称 = 值
- 改：UPDATE 表名称 SET 列名称 = 新值 WHERE 列名称 = 某值
- 查：SELECT 列名称（通配是*符号） FROM 表名称
ps:操作数据表是:ALTER TABLE。该语句用于在已有的表中添加、修改或删除列。
- ALTER TABLE table_name ADD column_name datatype
- ALTER TABLE table_name DROP COLUMN column_name
- ALTER TABLE table_name_old RENAME TO table_name_new

## Service的生命周期
``【参考】: ``
Service是运行在主线程中的，即主线程。
- startService() 开启Service，调用者退出后Service仍让存在。
- bindService() 开启Service，调用者退出后Service也随即退出。

Service的生命周期
- 只是startService的情况下，onCreate() -> onStartCommand() -> onDestroy()
- 只是bindService的情况下，onCreate() -> onBind() -> onUnBind() -> onDestroy()

## Activity之间的通信方式
``【参考】: ``
- Intent 
- 借助类的静态变量 
- 借助全局变量/Application 
- 借助外部工具
    * 借助 SharedPreference
    * 使用 Android 数据库 SQLite – 赤裸裸的使用File
    * Android 剪切板
- 借助Service

## SurfaceView和TextureView的区别
``【参考】: ``
相同点：都继承于View，可在独立线程绘制和渲染。

不同点：
* SurfaceView：嵌入视图层级内的绘制界面，是一种独立的View，更像是Window，不能做缩放、平移、画圆角等一般View形式操作；
* TextureView：更像是一般的View，可以做缩放、平移等View形式操作；

从性能和安全性角度出发，使用播放器优先选SurfaceView。
1.在android 7.0上系统surfaceview的性能比TextureView更有优势，支持对象的内容位置和包含的应用内容同步更新，平移、缩放不会产生黑边。 在7.0以下系统如果使用场景有动画效果，可以选择性使用TextureView
2.SurfaceView优点及缺点优点：可以在一个独立的线程中进行绘制，不会影响主线程，使用双缓冲机制，播放视频时画面更流畅
缺点：Surface不在View hierachy中，它的显示也不受View的属性控制，所以不能进行平移，缩放等变换，也不能放在其它ViewGroup中。SurfaceView 不能嵌套使用
TextureView优点及缺点
优点：支持移动、旋转、缩放等动画，支持截图
缺点：必须在硬件加速的窗口中使用，占用内存比SurfaceView高，在5.0以前在主线程渲染，5.0以后有单独的渲染线程

## onSaveInstanceState和onRestoreInstanceState调用时机
``【参考】: ``

```java
03-09 12:14:32.529 2298-2298/com.example.myapplication I/MY_TEST: onPause 
03-09 12:14:32.556 2298-2298/com.example.myapplication I/MY_TEST: onCreate2
03-09 12:14:32.557 2298-2298/com.example.myapplication I/MY_TEST: onStart2
03-09 12:14:32.557 2298-2298/com.example.myapplication I/MY_TEST: onResume2
03-09 12:14:32.981 2298-2298/com.example.myapplication I/MY_TEST: onSaveInstanceState 一个参数
03-09 12:14:32.981 2298-2298/com.example.myapplication I/MY_TEST: onStop

分割线-------------------------------------------------------------------------

03-09 12:15:28.715 2298-2298/com.example.myapplication I/MY_TEST: onPause2
03-09 12:15:28.763 2298-2298/com.example.myapplication I/MY_TEST: onCreate
03-09 12:15:28.764 2298-2298/com.example.myapplication I/MY_TEST: onStart
03-09 12:15:28.764 2298-2298/com.example.myapplication I/MY_TEST: onRestoreInstanceState
03-09 12:15:28.767 2298-2298/com.example.myapplication I/MY_TEST: onActivityResult 
03-09 12:15:28.767 2298-2298/com.example.myapplication I/MY_TEST: onResume 
03-09 12:15:29.141 2298-2298/com.example.myapplication I/MY_TEST: onStop2
03-09 12:15:29.141 2298-2298/com.example.myapplication I/MY_TEST: onDestroy2
```
1. 跳转过程中，先执行1的onpause，等等 onstop，等待2的onresume执行完之后，再执行1的onstop、ondestory
2. onSaveInstanceState每次隐藏activity都会在onpause之后执行(即使activity没有销毁也会执行)
3. onRestoreInstance 在 onstart 之后，onActivityResult 之前执行

## LeakCanary原理
``【参考】: ``
> 弱引用WeakReference
被强引用的对象就算发生 OOM 也永远不会被垃圾回收机回收;
被弱引用的对象，只要被垃圾回收器发现就会立即被回收;
被软引用的对象，具备内存敏感性，只有内存不足时才会 被回收，常用来做内存敏感缓存器;
虚引用则任意时刻都可能被回收，使用较少。

引用队列ReferenceQueue

> 我们常用一个 WeakReference reference = new WeakReference(activity);，这里我们创建了 一个 reference 来弱引用到某个 activity，当这个 activity 被垃圾回收器回收后，这个 reference 会被放入内部的 ReferenceQueue 中。也就是说，从队列 ReferenceQueue 取出 来的所有 reference，它们指向的真实对象都已经成功被回收了。

1. 利用 application.registerActivityLifecycleCallbacks(lifecycleCallbacks) 来监听整个生命周期内的 Activity onDestoryed 事件;
2. 当某个 Activity 被 destory 后，将它传给 RefWatcher 去做观测，确保其后续会被正常 回收;
3. RefWatcher 首先把 Activity 使用 KeyedWeakReference 引用起来，并使用一个 ReferenceQueue 来记录该 KeyedWeakReference 指向的对象是否已被回收;
4. AndroidWatchExecutor 会延迟 5 秒后，再开始检查这个弱引用内的 Activity 是否被正常 回收。判断条件是:若 Activity 被正常回收，那么引用它的 KeyedWeakReference 会被自 动放入 ReferenceQueue 中。
5. 判断方式是:先看 Activity 对应的 KeyedWeakReference 是否已经放入 ReferenceQueue 中;如果没有，则手动 GC:gcTrigger.runGc();;然后再一次判断 ReferenceQueue 是否已经含有对应的 KeyedWeakReference。若还未被回收，则认为可能发 生内存泄漏。
6. 利用 HeapAnalyzer 对 dump 的内存情况进行分析并进一步确认，若确定发生泄漏，则 利用 HeapAnalyzerService发送通知。
7. 弱引用与 ReferenceQueue 联合使用，如果弱引用关联的对象被回收，则会把这个弱引用 加入到 ReferenceQueue 中;通过这个原理，可以看出 removeWeaklyReachableReferences() 执行后，会对应删除 KeyedWeakReference 的数据。如果这个引用继续存在，那么就说明没 有被回收。
8. 为了确保最大保险的判定是否被回收，一共执行了两次回收判定，包括一次手动 GC 后 的回收判定。两次都没有被回收，很大程度上说明了这个对象的内存被泄漏了，但并不能 100% 保证;因此 LeakCanary是存在极小程度的误差的。

## Android系统为什么会设计ContentProvider
``【参考】: ``

在开发中，假如，A、B 进程有部分信息需要同步，这个时候怎么处理呢?设想这么一个场 景，有个业务复杂的 Activity 非常占用内存，并引发 OOM，所以，想要把这个 Activity 放到 单独进程，以保证 OOM 时主进程不崩溃。但是，两个整个 APP 有些信息需要保持同步，比 如登陆信息等，无论哪个进程登陆或者修改了相应信息，都要同步到另一个进程中去，这个 时候怎么做呢?

第一种:一个进程里面的时候，经常采用SharePreference 来做，但是SharePreference 不支持多进程，它基于单个文件的，默认是没有考虑同步互斥，而且，APP对SP对象做了缓存， 不好互斥同步，虽然可以通过 FileLock 来实现互斥，但同步仍然是一个问题。 第二种:基于 Binder 通信实现 Service 完成跨进程数据的共享，能够保证单进程访问数据， 不会有互斥问题，可是同步的事情仍然需要开发者手动处理。
第三种:基于Android提供的ContentProvider来实现，ContentProvider 同样基于Binder， 不存在进程间互斥问题，对于同步，也做了很好的封装，不需要开发者额外实现。

ContentProvider 只是 Android 为了跨进程共享数据提供的一种机制， 本身基于 Binder 实现，
在操作数据上只是一种抽象，具体要自己实现
ContentProvider 只能保证进程间的互斥，无法保证进程内，需要自己实现
ContentResolver 接口的 notifyChange 函数来通知那些注册了监控特定 URI 的 ContentObserver 对象，使得它们可以相应地执行一些处理。ContentObserver 可以通过 registerContentObserver 进行注册。
既然是对外提供数据共享，那么如何限制对方的使用呢?
android:exported 属性非常重要。这个属性用于指示该服务是否能够被其他应用程序组件调 用或跟它交互。如果设置为 true，则能够被调用或交互，否则不能。设置为 false 时，只有 同一个应用程序的组件或带有相同用户 ID 的应用程序才能启动或绑定该服务。

## Service和Activity通信
``【参考】: ``

Activity 调用 bindService (Intent service, ServiceConnection conn, int flags)方法，得到 Service 对象的一个引用，这样 Activity 可以直接调用到 Service 中的方法，如果要主动通知 Activity， 我们可以利用回调方法
Service 向 Activity 发送消息，可以使用广播，当然 Activity要注册相应的接收器。比如 Service 要向多个 Activity发送同样的消息的话，用这种方法就更好。

## OOM 是否可以try catch
``【参考】: ``
一般不适合这么处理
Java 中管理内存除了显式地 catch OOM 之外还有更多有效的方法:比如 SoftReference, WeakReference, 硬盘缓存等。
在 JVM 用光内存之前，会多次触发 GC，这些 GC 会降低程序运行的效率。
如果 OOM 的原因不是 try 语句中的对象(比如内存泄漏)，那么在 catch 语句中会继续抛出 OOM

## AlertDialog，Toast 对Activity生命周期的影响
``【参考】: ``
无论 Dialog 弹出覆盖页面，对 Activity 生命周期没有影响，只有再启动另外一个 Activity 的时候才会进入 onPause 状态，而不是想象中的被覆盖或者不可见，同时通过 AlertDialog 源码或者 Toast 源码我们都可以发现它们实现的原理都是 windowmanager.addView();来添加的， 它们都是一个个 view ,因此不会对 activity 的生命周 期有任何影响。

## HTTP与TCP的区别和联系
``【参考】: ``
TCP 连接
 
手机能够使用联网功能是因为手机底层实现了 TCP/IP 协议，可以使手机终端通过无线网络，建立 TCP 连接。TCP 协议可以对上层网络提供接口，使上层网络数据的传输建立在“无差别” 的网络之上。

建立起一个 TCP 连接需要经过“三次握手”:
- 第一次握手:客户端发送 syn 包(syn=j)到服务器，并进入 SYN_SEND 状态，等待服务器确认;
- 第二次握手:服务器收到 syn 包，必须确认客户的 SYN(ack=j+1)，同时自己也发送一个SYN 包(syn=k)，即 SYN+ACK 包，此时服务器进入 SYN_RECV 状态;
- 第三次握手:客户端收到服务器的 SYN+ACK 包，向服务器发送确认包 ACK(ack=k+1)，此包发送完毕，客户端和服务器进入 ESTABLISHED 状态，完成三次握手。

握手过程中传送的包里不包含数据，三次握手完毕后，客户端与服务器才正式开始传递数据。理想状态下，TCP连接一旦建立，在通信双方中的任何一方主动关闭连接之前，TCP连接都将被一直保持下去。断开连接时服务器和客户端均可以主动发起断开TCP连接的请求，断开过程需要经过“四次挥手”。

HTTP 连接

HTTP 连接最显著的特点是客户端发送的每次请求都需要服务器回送响应，在请求结束后，
理想状态下，TCP 连接一旦建立，在通信双方中的任何一方主动关闭连 接之前，TCP 连接
都将被一直保持下去。断开连接时服务器和客户端均可以主动发起断开 TCP 连接的请求，
断开过程需要经过“四次握手”(过程就不细写 了，就是服务器和客户端交互，最终确定断
开)
 HTTP 协议即超文本传送协议(Hypertext Transfer Protocol )，是 Web 联网的基础，也是手
 机联网常用的协议之一，HTTP 协议是建立在 TCP 协议之上的一种应用。
 会主动释放连接。从建立连接到关闭连接的过程称为“一次连接”。
- 1)在 HTTP 1.0 中，客户端的每次请求都要求建立一次单独的连接，在处理完本次请求后，
 就自动释放连接。
- 2)在 HTTP 1.1 中则可以在一次连接中处理多个请求，并且多个请求可以重叠进行，不需要
  等待一个请求结束后再发送下一个请求。
由于 HTTP 在每次请求结束后都会主动释放连接，因此 HTTP 连接是一种“短连接”，要保持客户端程序的在线状态，需要不断地向服务器发起连接请求。通常的 做法是即时不需要获
得任何数据，客户端也保持每隔一段固定的时间向服务器发送一次“保持连接”的请求，服务器在收到该请求后对客户端进行回复，表明知道客 户端“在线”。若服务器长时间无法收到
客户端的请求，则认为客户端“下线”，若客户端长时间无法收到服务器的回复，则认为网络已经断开。

TPC/IP 协议是传输层协议，主要解决数据如何在网络中传输，而 HTTP 是应用层协议，主要 解决如何包装数据。TCP 协议对应于传输层，而 HTTP 协议对应于应用层，从本质上来说， 二者没有可比性。Http 协议是建立在 TCP 协议基础之上的，当浏览器需要从服务器获取网 页数据的时候，会发出一次 Http 请求。Http 会通过 TCP 建立起一个到服务器的连接通道。 简单地说，当一个网页打开完成后，客户端和服务器之间用于传输 HTTP 数据的 TCP 连接不会关闭，如果客户端再次访问这个服务器上的网页，会继续使用这一条已经建立的连接
 Keep-Alive 不会永久保持连接，它有一个保持时间，可以在不同的服务器软件(如 Apache)
 中设定这个时间。Http 是无状态的短连接，而 TCP 是有状态的长连接。

 ## 为什么要用多进程?有哪些方式?怎么使用多进程?
``【参考】: ``
 那么多进程应该能为我们带来什么呢?
我们都知道，android 平台对应用都有内存限制，其实这个理解有点问题，应该是说 android 平台对每个进程有内存限制，比如某机型对对进程限制是 24m，如果应用有两个进程，则该应该的总内存限制是 2*24m。使用多进程就可以使得我们一个 apk 所使用的内存限制加大几倍。 所以可以借此图片平台对应用的内存限制，比如一些要对图片、视频、大文件进程处理的好内存的应用可以考虑用多进程来解决应用操作不流畅问题。

开启多进程模式:
在 Android 中使用多进程只有一种方法,那就是在 AndroidManifest 中给四大组件 (Activity,Service,Receiver,ContentProvider)指定 android:process 属性.除此之外没有其他的办 法,也就是说我们无法给一个线程活一个实体类指定其运行时所在的进程.其实还有另一种非常规的多进程方法,那就是通过 JNI 在 native 层去 fork一个新的进程,但这种方法属于特殊情 况,并不是常用的创建多进程的方式,所以我们也暂不考虑这种情况
进程名以":"开头的进程属于当前应用的私有进程,其他应用的组件不可以和它跑在同一个进 程中,而进程名不以":"开头的进程属于全局进程,其他应用通过 ShareUID 方式可以和它跑在 同一个进程中.

用多进程的好处与坏处
好处:
1)分担主进程的内存压力。 当应用越做越大，内存越来越多，将一些独立的组件放到不同的进程，它就不占用主进程的 内存空间了。当然还有其他好处，有心人会发现 2)使应用常驻后台，防止主进程被杀守护进程，守护进程和主进程之间相互监视，有一方 被杀就重新启动它。Android 后台进程里有很多应用是多个进程的，因为它们要常驻后台，特别是即时通讯或者社交应用，不过现在多进程已经被用烂了。
典型用法是在启动一个不可见的轻量级私有进程，在后台收发消息，或者做一些耗时的事情， 或者开机启动这个进程，然后做监听等。

坏处:消耗用户的电量。
多占用了系统的空间，若所有应用都这样占用，系统内存很容易占满而导致卡顿。 应用程序架构会变得复杂，因为要处理多进程之间的通信。这里又是另外一个问题了。
多进程的缺陷 进程间的内存空间是不可见的。开启多进程后，会引发以下问题:
1)Application 的多次重建。 2)静态成员的失效。 3)文件共享问题。 4)断点调试问题。

## viewstub可以多次inflate么?多次inflate会怎样?
``【参考】: ``
在使用 viewstub 的时候要注意一点，viewstub 只能 inflate 一次，而且 setVisibility 也会间接的调用到 inflate，重复 inflate 会抛出异常: java.lang.IllegalStateException:ViewStub must have a non-null ViewGroup viewParent。

解决方法为设置一个 Boolean 类型的变量，标记 viewstub 是否已经 inflate，如果 viewstub 还未 inflate 则执行初始化操作，反之则不进行操作。其中要使用 ViewStub 中的 OnInflateListener()监听事件来判断是否已经填充,从而保证 viewstub 不重复的 inflate。

解决方法:
1.定义 boolean 变量和 ViewStub boolean isInflate = false; ViewStub mViewStub;

2.初始化 ViewStub，并为 ViewStub 添加 OnInflateListener()监听事件
```java
mViewStub = (ViewStub)findViewById(R.id.viewstub_match_single);
mViewStub.setOnInflateListener(new OnInflateListener() { @Override
public void onInflate(ViewStub stub, View inflated) {
isInflate = true; }
});
```
 3.填充 ViewStub
 ```java
private void initViewStub(){//填充 ViewStub 的方法
    if(!isInflate){//如果没有填充则执行 inflate 操作
        View view = stubMatchSingle.inflate();
        //初始化 ViewStub 的 layout 里面的控件
        TextView mTv = (TextView) view.findViewById(R.id.txt_url); mTv.setOnClickListener(this);
    } 
}
 ```

 ## 如何获取一个图片的宽高
 ``【参考】: ``
 android 在不加载图片的前提下获得图片的宽高
 ```java
public static int[] getImageWidthHeight(String path){
BitmapFactory.Options options = new BitmapFactory.Options();
    /**
    * 最关键在此，把 options.inJustDecodeBounds = true;
    * 这里再 decodeFile()，返回的 bitmap 为空，但此时调用 options.outHeight 时，已经
    包含了图片的高了 */
    options.inJustDecodeBounds = true; 
    Bitmapbitmap=BitmapFactory.decodeFile(path,options);// 此时返回的bitmap为null /**
    /*
    *options.outHeight 为原始图片的高
    */
    return new int[]{options.outWidth,options.outHeight};
}
 ```

 通过 BitmapFactory 从不同位置获取 Bitmap
* 1.资源文件(drawable/mipmap/raw)
BitmapFactory.decodeResource(getResources(), R.mipmap.slim_lose_weight_plan_copenhagen,options);
* 2.资源文件(assets)
InputStream is = getActivity().getAssets().open("bitmap.png"); BitmapFactory.decodeStream(is);
* 3.内存卡文件
bitmap = BitmapFactory.decodeFile("/sdcard/bitmap.png");
* 4.网络文件
bitmap = BitmapFactory.decodeStream(is);
可根据 BitmapFactory 获取图片时传入 option，通过上述方法获取图片的宽高

## onWindowFocusChanged 执行时机
``【参考】: ``
真正的 visible 时间点是 onWindowFocusChanged()函数被执行时，当前窗体得到
 或失去焦点的时候的时候调用。这是这个活动是否是用户可见的最好的指标。