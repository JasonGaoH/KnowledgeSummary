# ConcurrentHashMap
之前分析过HashMap的一些实现细节，[HashMap的原理与实现](./HashMap的原理与实现.md), 今天我们从源码角度来看看ConcurrentHashMap是如何实现线程安全的，其实网上这类文章分析特别多，秉着”纸上得来终觉浅，绝知此事要躬行“的原则，我们尝试自己去分析下，希望这样对于ConcurrentHashMap有一个更深刻的理解。

### 为什么说HashMap线程不安全，而ConcurrentHashMap就线程安全
其实ConcurrentHashMap在Android开发中使用的场景并不多，但是ConcurrentHashMap为了支持多线程并发这些优秀的设计却是最值得我们学习的地方，往往”ConcurrentHashMap是如何实现线程安全“这类问题却是面试官比较喜欢问的问题。

首先，我们尝试用代码模拟下HashMap在多线程场景下会不安全，如果把这个场景替换成ConcurrentHashMap会不会有问题。


### JDK8的ConcurrentHashMap文档提炼
 * ConcurrentHashMap支持检索的完全并发和更新的高预期并发性,这里的说法很有意思检索支持完全并发，更新则支持高预期并发性，因为它的检索操作是没有加锁的，实际上检索也没有必要加锁。实际上ConcurrentHashMap和Hashtable在不考虑实现细节来说，这两者完全是可以互相操作的,Hashtable在get，put，remove等这些方法中全部加入了synchronized，这样的问题是能够实现线程安全，但是缺点是性能太差，几乎所有的操作都加锁的，但是ConcurrentHashMap的检测操作却是没有加锁的。
 * ConcurrentHashMap检索操作(包括get)通常不会阻塞，因此可能与更新操作(包括put和remove)重叠。
 * ConcurrentHashMap跟Hashtable类似但不同于HashMap，它不可以存放空值，key和value都不可以为null。

 印象中一直以为ConcurrentHashMap是基于Segment分段锁来实现的，之前没仔细看过源码，一直有这么个错误的认识。ConcurrentHashMap是基于Segment分段锁来实现的，这句话也不能说不对，加个前提条件就是正确的了，ConcurrentHashMap从JDK1.5开始随java.util.concurrent包一起引入JDK中，在JDK8以前，ConcurrentHashMap都是基于Segment分段锁来实现的，在JDK8以后，就换成synchronized和CAS这套实现机制了。

JDK8中ConcurrentHashMap摒弃了segment锁，直接将hash桶的头结点当做锁。

[从ConcurrentHashMap的演进看Java多线程核心技术](http://www.jasongj.com/java/concurrenthashmap/)

[ConcurrentHashMap源码分析(JDK8)get/put/remove方法分析](https://www.jianshu.com/p/5bc70d9e5410)

[探索 ConcurrentHashMap 高并发性的实现机制](https://www.ibm.com/developerworks/cn/java/java-lo-concurrenthashmap/index.html)

[Java7/8 中的 HashMap 和 ConcurrentHashMap 全解析](http://www.importnew.com/28263.html)

[Java HashMap工作原理及实现](https://yikun.github.io/2015/04/01/Java-HashMap%E5%B7%A5%E4%BD%9C%E5%8E%9F%E7%90%86%E5%8F%8A%E5%AE%9E%E7%8E%B0/)

[Java 8：HashMap的性能提升](http://www.importnew.com/14417.html)