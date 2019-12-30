RxJava+Retrofit进行网络请求; 
Header里如何添加通用参数，
Response里如何解析，只返回业务层result，不需要每个Bean都有code

设计模式：工厂  建造者  装饰者

mvp模式的了解和介绍

对RecyclerView的了解，踩过哪些坑？

margin 和padding的区别
MeasureSpeac.EXACTLY/UN.../ATMOLST -> wrap_content/match_parent
绘制流程？控件的margin，是在measure还是layout，还是draw过程中处理？

方法数超过65535怎么解决？

子线程中进行UI操作方法有哪些？ Handler的post()方法；View的post()方法；Activity的runOnUiThread()方法

事件分发
String, StringBuffer, StringBuilder的区别；
String: 定长，字符串操作会产生新的String对象；
StringBuffer：线程安全，有synchronized修饰，不定长，默认容量是16个字节+传入字符串长度，可扩展，操作不会产生新的对象；
StringBuilder：线程不安全，其他同StringBuffer；

final 修饰符的作用
>final 可以修饰类、变量和方法。修饰类代表这个类不可被继承。修饰变量代表此变量不可被改变。修饰方法表示此方法不可被重写 (override)。


Java：
* Object有哪些方法，每个方法的用处？wait  notify  hashCode  finalize  getClass  toString clone  equals()延伸：wait和sleep区别
* Java内存模型，每块内存存放哪些东西？ String的常量池存放在哪里？
* Java垃圾回收的几个算法？有哪些垃圾收集器，优缺点是什么？比如G1收集器相对于CMS有哪些优缺点？
* 检测内存泄漏用什么工具？内存泄露是怎么监测的?LeakCanery的原理？
* 内存泄漏的场景？
    * 内部类持有外部类的引用原理？
    * 四种引用方式；
* 关键字volatile使用场景？（不会volatile，可问对Java并发的3个重要特性：原子性、可见性、有序性的理解？）

http://www.cnblogs.com/dolphin0520/p/3920373.html

7.怎么解决死锁问题？
>某个任务在等待另一个任务，而后者又等待别的任务，这样一直下去，直到这个链条上的任务又在等待第一个任务释放锁。这得到了一个任务之间互相等待的连续循环，没有哪个线程能继续。这被称之为死锁。当以下四个条件同时满足时，就会产生死锁：
(1) 互斥条件。任务所使用的资源中至少有一个是不能共享的。
(2) 任务必须持有一个资源，同时等待获取另一个被别的任务占有的资源。
(3) 资源不能被强占。
(4) 必须有循环等待。一个任务正在等待另一个任务所持有的资源，后者又在等待别的任务所持有的资源，这样一直下去，直到有一个任务在等待第一个任务所持有的资源，使得大家都被锁住。
要解决死锁问题，必须打破上面四个条件的其中之一。在程序中，最容易打破的往往是第四个条件。

6.类加载机制？双亲委派模型的优点？
6.说说泛型，泛型原理，为什么可以用T；
7.反射机制，及其优劣处；
8.Override和Overload的含义去区别？（进阶：Java静态分派和动态分派？这两个谁是单分派，谁是多分派？）
9.说说对注解的了解，如何设计Butterknife？
9.用到了哪些设计模式？知道装饰者、观察者、组合模式？

Android
1.Activity的四种启动方式
2.绘制流程？控件的margin，是在measure还是layout，还是draw过程中处理？
3.事件分发机制？onTouch和onTouchEvent是什么区别？如果我重写了onTouch和onClick，它们的调用顺序是怎样的？什么时候会不调用onClick？

4.LocalBroadCast实现原理
>本地广播是通过LocalBroadcastManager内置的Handler来实现的，只是利用了IntentFilter的match功能，至于BroadcastReceiver 换成其他接口也无所谓，顺便利用了现成的类和概念而已。在register()的时候保存BroadcastReceiver以及对应的IntentFilter，在sendBroadcast()的时候找到和Intent对应的BroadcastReceiver，然后通过Handler发送消息，触发executePendingBroadcasts()函数，再在后者中调用对应BroadcastReceiver的onReceive()方法。

6.SurfaceView用过么？双缓冲的原理是什么？
6.apk如何瘦身？
7.方法数超过65535怎么解决？multidex何时将包合并？




-------------------------------------------------------------------

1.不可变对象，如string

2.apk如何瘦身

3.一张图片内存大小

4.如何检查没有使用的资源

5.multidex何时将包合并

6.MVP相比MVC的优势

7.混合开发

8.UI层优化：如何解决过度绘制、滑动卡顿等

9.内存优化：

10.插件化相关

11.异步加载
          

12.动态加载

13.个性化UI时有哪些需要注意的，比较性能方面







面试题：
1、JNI：
1）NewStringUtf 、NewString、GetStringUtfChars、GetStringChars 区别？
从 返回值，释放方式方面解答。
NewString
jstring NewString(JNIEnv *env, const jchar *unicodeChars, jsize len);
Constructs a new java.lang.String object from an array of Unicode characters.

NewStringUTF
jstring NewStringUTF(JNIEnv *env, const char *bytes);
Constructs a new java.lang.String object from an array of characters in modified UTF-8 encoding.

GetStringChars
const jchar * GetStringChars(JNIEnv *env, jstring string, jboolean *isCopy);
Returns a pointer to the array of Unicode characters of the string. This pointer is valid until ReleaseStringchars() is called.

GetStringUTFChars
const char * GetStringUTFChars(JNIEnv *env, jstring string, jboolean *isCopy);
Returns a pointer to an array of bytes representing the string in modified UTF-8 encoding. This array is valid until it is released by ReleaseStringUTFChars().
If isCopy is not NULL, then *isCopy is set to JNI_TRUE if a copy is made; or it is set to JNI_FALSE if no copy is made.


2）modified utf-8 encoding 是什么意思？

3）如果Java层传入的字符串是Unicode的扩展字符，比如emoji表情，一个字符占4个字节的，JNI层将这个jstring转换成char会遇到什么问题？怎么解决呢？
坑：不能用JNI的GetStringUTFChars，要用Java的反射，将jstring转换成char.

3）native的代码中，遇到过哪些异常代码会使程序崩溃？ 
  （比如CallObjectMethod() null的参数会崩）

4）ndk-build的参数了解哪些？

2、Android
##Activity的四种启动方式；
     standard, singleTop, singleTask, 

##android使用xml文件的优缺点；
优点：在xml中写布局，实现UI和功能分开，方便机型适配、APP国际化；
缺点：解析xml文件耗时；

##Handler机制
     Handler依附于创建的线程；Looper、Message、Handler的关系；

3）子线程中进行UI操作方法有哪些？ Handler的post()方法；View的post()方法；Activity的runOnUiThread()方法

4）图片压缩有哪些方法？

5）ANR：Service、广播造成的ANR如何解决；
     5s UI线程没有响应；10s 没有接收到广播；

     解决ANR方法：
     1）不要在UI线程或影响UI渲染的地方做耗时操作；
     2）可以在/data/anr/traces.txt中，查看ANR日志；

6）IntentService和Service区别

6）OOM：如何解决；
http://www.cnblogs.com/xsmhero/p/3566890.html

7）四种引用方式；

# Android内存结构
#JVM垃圾回收机制
#内存泄露是怎么监测的?

#Android中的同步方式有哪些？
同步方法、同步代码块、原子操作

3、网络

4、Java
HashMap和HashTable的区别；
1）线程安全：HashMap不安全，HashTable安全，因为HashTable有Synchronized修饰；
2）速度：HashMap快一点；
3）允许空键：HashMap允许空键；

String, StringBuffer, StringBuilder的区别；
String: 定长，字符串操作会产生新的String对象；
StringBuffer：线程安全，有synchronized修饰，不定长，默认容量是16个字节+传入字符串长度，可扩展，操作不会产生新的对象；
StringBuilder：线程不安全，其他同StringBuffer；

说说泛型，为什么可以用T；
反射机制，及其优劣处；
设计模式：代理模式；
了解到的加密算法有哪些？比如SHA1和MD5能解释下原理么？
对称加密和非对称加密？