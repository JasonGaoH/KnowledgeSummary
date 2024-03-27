## Java 系列集合源码分析

工欲善其事，必先利其器。平时我们用的最多的API就是Java中各式各样的集合类了，要想运用得当，必须要对集合中一些实现细节有所了解。

[ArrayList和Vector的区别](./从源码角度分析ArrayList和Vector的区别.md) 主要介绍了ArrayList和Vector的差异，Vector因为其自身性能的问题，我们现在几乎不用这个集合类了，这里可以简单了解下。
  
[ArrayList和LinkedList的区别](./从源码角度解析ArrayList和LinkedList的区别.md) 主要介绍了ArrayList和LinkedList的差异，这两个集合可以说是我们开发过程中遇到最多的集合类，我们需要学习两者在实现上的差异性来灵活地根据不同的场景调整集合的使用策略。
  
[关于HashMap你需要知道的一些细节](./关于HashMap你需要知道的一些细节.md) 这篇文章主要介绍了HashMap中一些实现细节，如何解决hash碰撞，如何扩容等问题。
  
[ConcurrentHashMap是如何保证线程安全的](./ConcurrentHashMap是如何保证线程安全的.md)，ConcurrentHashMap在Android 的开发中用的很少，但是在面试过程中面试官经常会结合多线程的相关知识来考验你对于它的理解。
  
[Java中的红黑树解析](./我画了近百张图来理解红黑树.md)这篇文章最早是由我在公司内部的一个分享转换过来的，之前发在掘金平台，仅这一篇文章就收获了四百多的点赞，为了将红黑树讲清楚，说了很多张图，理解红黑树，这篇文章就够了。