
#### 应用启动流程
冷启动和热启动相比，冷启动多了一个创建APP 进程的过程，热启动由于进程已经存在，没有这个创建的过程。

一般来说，冷启动包括了以下内容：


1、启动进程
点击图标发生在Launcher应用的进程，startActivity()函数最终是由Instrumentation通过Android的Binder跨进程通信机制 发送消息给 system_server 进程；
在 **system_server** 中，启动进程的操作在ActivityManagerService。AMS发现ProcessRecord不存在时，就会执行Process.start()，最终是通过 socket 通信告知 Zygote 进程 fork 子进程（app进程）


2、开启主线程
app进程创建后，首先是反射调用android.app.ActivityThread类的main方法，main()函数里会初始化app的运行时环境：创建 ApplicationThread，Looper，Handler 对象，并开启主线程消息循环Looper.loop()。


3、创建并初始化 Application和Activity
ActivityThread的main()调用 ActivityThread#attach(false) 方法进行 Binder 通信，通知system_server进程执行 ActivityManagerService#attachApplication(mAppThread) 方法，用于初始化Application和Activity。
在system_server进程中，ActivityManagerService#attachApplication(mAppThread)里依次初始化了Application和Activity，分别有2个关键函数：
- thread#bindApplication() 方法通知主线程Handler 创建 Application 对象、绑定 Context 、执行 Application#onCreate() 生命周期
- mStackSupervisor#attachApplicationLocked() 方法中调用 ActivityThread#ApplicationThread#scheduleLaunchActivity() 方法，进而通过主线程Handler消息通知创建 Activity 对象，然后再调用 mInstrumentation#callActivityOnCreate() 执行 Activity#onCreate() 生命周期


4、布局&绘制
