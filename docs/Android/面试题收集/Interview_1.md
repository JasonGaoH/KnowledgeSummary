基础

1. 讲一下Android应用程序启动过程
2. Android中进程间的通信有哪些方法
3. view的工作原理及measure、layout、draw流程，哪一个可以放在子线程中执行
4. 讲一下Android View的事件分发机制
5. 如果控件内部卡顿你如何去解决并优化
6. listview和recycleview缓存机制
7. handler机制是什么
8. 讲一下IntentService
9. 讲一下Bitmap加载，三级缓存机制，如何实现一个LRUCache
10. 项目中做过哪些android性能优化，怎么做的？
11. Java的内存模型，每个部分的特点和作用
12. 讲一下你理解的GC
13. 讲一下对内存泄露的理解，常见的内存泄漏有哪些，怎么解决的
14. 为什么要使用多线程？多线程需要注意那些问题
15. 导致线程不安全的原因有哪些，怎么解决
16. lock和synchronize有什么区别
17. 讲一下乐观锁、悲观锁及使用场景
18. 讲一下hashmap的实现，hashtable和hashmap的区别是什么，hashtable为什么会被弃用，ConcurrentHashmap的实现，segment的概念、concurrenthashmap高效的原因是什么
19. 使用过那些三方库，有没有阅读过一些源码，找一个你认为的技术亮点介绍一下
20. Broadcast的分类？有序，无序？粘性，非粘性？本地广播？
>广播可以分为有序广播、无序广播、本地广播、粘性广播。其中无序广播通过sendBroadcast(intent)发送，有序广播通过sendOrderedBroadcast(intent)发送。

>有序广播
(1) 有序广播可以用priority来调整优先级   取值范围-1000~+1000，默认为0，数值越大优先级越高，优先级越高越优先获得广播响应。
(2) abortBroadcast()可来终止该广播的传播，对更低优先级的屏蔽，注意只对有序广播生效。
(3) 有序广播在传播数据中会发生比如setResultData()，getResultData()，在传播过程中，可以从新设置数据

>关于本地广播，可以查看这篇文章。总的来说，本地广播是通过LocalBroadcastManager内置的Handler来实现的，只是利用了IntentFilter的match功能，至于BroadcastReceiver 换成其他接口也无所谓，顺便利用了现成的类和概念而已。在register()的时候保存BroadcastReceiver以及对应的IntentFilter，在sendBroadcast()的时候找到和Intent对应的BroadcastReceiver，然后通过Handler发送消息，触发executePendingBroadcasts()函数，再在后者中调用对应BroadcastReceiver的onReceive()方法。

>粘性消息：粘性消息在发送后就一直存在于系统的消息容器里面，等待对应的处理器去处理，如果暂时没有处理器处理这个消息则一直在消息容器里面处于等待状态，粘性广播的Receiver如果被销毁，那么下次重建时会自动接收到消息数据。(在 android 5.0/api 21中deprecated,不再推荐使用，相应的还有粘性有序广播，同样已经deprecated)

进阶

1. 讲一下classloader有哪些，如何自定义一个classloader
2. 讲一下双亲委派机制
3. Android中热修复如何实现
4. Android中插件化如何实现
5. Bunder的机制和原理是什么
6. JVM内存分配策略，垃圾回收标记算法，垃圾回收策略
7. Minor GC、Major GC和Full GC之间的区别
8. JsBridge的实现
9. 讲一下MVC，MVP，MVVM的理解

扩展

1. 是否使用过kotlin，说说你的感受
2. RN和weex有没有接触过，实现原理是什么
3. 前端有没有做过，做过什么项目，使用的什么框架
4. 服务端有没有了解过，使用过什么框架
5. 其他语言是否接触过，有没有过一些项目开发

项目

1. 介绍一下目前项目的整体架构，你认为现在的架构中是否存在问题，什么问题，如何解决
2. 项目中使用的一些三方库，有没有深入看过源码，挑一个你认为设计的比较好的讲一下，为什么你认为这个地方设计的比较好
3. 从业务和技术实现方案两个方面，讲一个你主导过或者是作为核心人员参与过的一个项目，这个项目中的亮点是什么，针对项目进行一些提问
4. 简历中挑选几个感兴趣点或项目，询问一下项目的技术实现
5. 讲一下工作中遇到过的困难，怎么解决的，这件事对你的成长是什么
6. 为什么离职，未来1-2年的职业规划是什么，你期待新的岗位能给你提供什么