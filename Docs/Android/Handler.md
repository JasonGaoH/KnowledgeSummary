Handler如何在handleMessage方法拦截之前发出的message

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

Android中为什么主线程不会因为Looper.loop()里的死循环卡死？
* [主线程的工作原理](https://haldir65.github.io/2016/10/12/2016-10-12-How-the-mainThread-work/)
  > 这篇文章笔记有新意的思考点是：在2.2版本以前，这套机制是用我们熟悉的线程的wait和notify 来实现的,之前的Android版本用的是Java的线程wait和notify。

  >Android应用程序的主线程在进入消息循环过程前，会在内部创建一个Linux管道（Pipe），这个管道的作用是使得Android应用程序主线程在消息队列为空时可以进入空闲等待状态，并且使得当应用程序的消息队列有消息需要处理时唤醒应用程序的主线程。