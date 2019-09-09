> Android中面试题总结主要用于收录一些零散的面试题，这些面试题因为知识点比较零散，所以准备把这部分整理做个总结。对于有的知识点考虑篇幅很大，不会放在这篇文章中，会作为一篇独立的文章来总结放在最外面的REDAME目录中。

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
> 如果时长连接，那么一个 socket(tcp)可能发送和接收多次请求，那么如何判断每次的 响应已经接收?
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


## onStartCommand 的几种模式

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