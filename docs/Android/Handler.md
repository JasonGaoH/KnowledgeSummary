Handler 为什么会泄漏

Handler如何在handleMessage方法拦截之前发出的message

> 通常会伴随着一个耗时的后台线程（例如从网络拉取图片）一起出现，这个后台线程在任务执行完毕（例如图片下载完毕）之后，通过消息机制通知Handler，然后Handler把图片更新到界面。然而，如果用户在网络请求过程中关闭了Activity，正常情况下，Activity不再被使用，它就有可能在GC检查时被回收掉，但由于这时线程尚未执行完，而该线程持有Handler的引用（不然它怎么发消息给Handler？），这个Handler又持有Activity的引用，就导致该Activity无法被回收（即内存泄露），直到网络请求结束（例如图片下载完毕）。另外，如果你执行了Handler的postDelayed()方法，该方法会将你的Handler装入一个Message，并把这条Message推到MessageQueue中，那么在你设定的delay到达之前，会有一条MessageQueue -> Message -> Handler -> Activity的链，导致你的Activity被持有引用而无法被回收。

``` java

/**
* Handle system messages here.
*/
public void dispatchMessage(Message msg) {
//首先检查msg是否设置了回调
    if (msg.callback != null) {
        handleCallback(msg);
    } else {
        if (mCallback != null) {
        //重点在这里，callback的handleMessage()方法的返回值决定了是否拦截消息
        //重点在这里，callback的handleMessage()方法的返回值决定了是否拦截消息
        //重点在这里，callback的handleMessage()方法的返回值决定了是否拦截消息
            if (mCallback.handleMessage(msg)) {
                return;
            }
        }
        handleMessage(msg);
    }
}

```

*  Handler
   *  Handler如何在handleMessage方法拦截之前发出的message
   *  HandlerThread的原理
   *  IntentService的实现原理
   *  Handler实现机制（很多细节需要关注：如线程如何建立和退出消息循环等等
   *  Handler发消息给子线程，looper怎么启动
   *  Handler postDelay这个延迟是怎么实现的
   *  MessageQueue.IdleHandler 你了解嘛
   *  Handler 造成内存泄漏，是什么东西造成的泄漏

[Android 消息处理机制（Looper、Handler、MessageQueue,Message）](https://www.jianshu.com/p/02962454adf7)

Android中为什么主线程不会因为Looper.loop()里的死循环卡死？
* [主线程的工作原理](https://haldir65.github.io/2016/10/12/2016-10-12-How-the-mainThread-work/)
  > 这篇文章笔记有新意的思考点是：在2.2版本以前，这套机制是用我们熟悉的线程的wait和notify 来实现的,之前的Android版本用的是Java的线程wait和notify。

  >Android应用程序的主线程在进入消息循环过程前，会在内部创建一个Linux管道（Pipe），这个管道的作用是使得Android应用程序主线程在消息队列为空时可以进入空闲等待状态，并且使得当应用程序的消息队列有消息需要处理时唤醒应用程序的主线程。

  这个主线程的休眠以及唤醒操作是基于Linux的epoll机制来实现的。

  在MessageQueue.java的next方法中：

```java
Message next() 
       
        final long ptr = mPtr;
        if (ptr == 0) {
            return null;
        }
        int pendingIdleHandlerCount = -1; // -1 only during first iteration

        //nextPollTimeoutMillis 表示nativePollOnce方法需要等待nextPollTimeoutMillis 
        //才会返回
        int nextPollTimeoutMillis = 0;
        for (;;) {
            if (nextPollTimeoutMillis != 0) {
                Binder.flushPendingCommands();
            }
            //读取消息，队里里没有消息有可能会堵塞，两种情况该方法才会返回(代码才能往下执行)
            //一种是等到有消息产生就会返回,
            //另一种是当等了nextPollTimeoutMillis时长后，nativePollOnce也会返回
            nativePollOnce(ptr, nextPollTimeoutMillis);
            //nativePollOnce 返回之后才能往下执行
            synchronized (this) {
                // Try to retrieve the next message.  Return if found.
                final long now = SystemClock.uptimeMillis();
                Message prevMsg = null;
                Message msg = mMessages;
                if (msg != null && msg.target == null) {
                    // 循环找到一条不是异步而且msg.target不为空的message
                    do {
                        prevMsg = msg;
                        msg = msg.next;
                    } while (msg != null && !msg.isAsynchronous());
                }
                if (msg != null) {
                    if (now < msg.when) {
                       // 虽然有消息，但是还没有到运行的时候，像我们经常用的postDelay,
                       //计算出离执行时间还有多久赋值给nextPollTimeoutMillis，
                       //表示nativePollOnce方法要等待nextPollTimeoutMillis时长后返回
                        nextPollTimeoutMillis = (int) Math.min(msg.when - now, Integer.MAX_VALUE);
                    } else {
                        // 获取到消息
                        mBlocked = false;
                       //链表一些操作，获取msg并且删除该节点 
                        if (prevMsg != null) 
                            prevMsg.next = msg.next;
                        } else {
                            mMessages = msg.next;
                        }
                        msg.next = null；
                        msg.markInUse();
                        //返回拿到的消息
                        return msg;
                    }
                } else {
                    //没有消息，nextPollTimeoutMillis复位
                    nextPollTimeoutMillis = -1;
                }
                .....
                .....
              
    }
```
nativePollOnce()很重要，是一个native的函数，``nativePollOnce``最终调用到的是：

> /frameworks/base/core/jni/android_os_MessageQueue.cpp中的``pollOnce``方法，pollOnce内部又是调用``pollInner(int timeoutMillis)``方法。

pollInner中就是调用epoll_wait，epoll_wait就是使用的是Linux中epoll机制，关于epoll的机制可以看下这篇关于epoll的文章分析。[epoll的原理和实现](https://tqr.ink/2017/10/05/implementation-of-epoll/)。

nativePollOnce这里会读取消息，队里里没有消息有可能会堵塞，这个时候会让出处理器，当有消息来了或者当设置的timeout时间到了就可以往下执行。