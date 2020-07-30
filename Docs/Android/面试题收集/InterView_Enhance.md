节选自小专栏中的《Android 面试指南》

## OPPO

### 一面
- 自我介绍，聊聊简历上写的项目
- MVC、MVP的区别
- RxJava的线程切换原理
- ListView的卡顿优化、资讯流懒加载如何实现
- 序列化前后对象有何区别
- Binder通信原理
- 插件化开发流程，插件化优势，插件化开发中遇到的问题以及如何解决的

### 二面
- 自我介绍，项目介绍
- Glide的优势，它是如何绑定生命周期的
- ListView如何优化，ListView和RecyclerView的区别，二者的缓存逻辑
- 提高app安全性的方法
- View的绘制流程
- 插件化原理

### 三面
- 自我介绍，项目介绍
- 为什么选用插件化，插件化框架的比较，梳理插件化的架构
- App的启动流程
- 自己的优势和劣势

## 腾讯视频

### 一面
- 自我介绍，项目介绍
- 如何处理App启动流程优化
- View的性能优化
- MVC到MVP的架构重构流程

### 二面
- 自我介绍，项目介绍
- 为什么用Glide，它的优势
- HashMap的原理，ConCurrentHashMap的原理
- View的绘制流程
- Handler的工作原理
- LeakCanary原理，内存泄露的分析流程
- 进程间通信方式，Binder原理
- 垃圾回收原理以及回收策略
- Java中的强、软、弱、虚引用
- 算法：排序，要求时间复杂度是O(n)，例如1，4，5，2，7，6，9。如果数列是10，102，99，10000，825，1003030如何排序
  - 对于数值不大的可以构造一个数组来实现O(n)的排序

### 三面
- View的性能优化，页面卡顿的原因
- TCP、IP各自所在的网络层次，IP报文中的内容
- https的对称加密和非对称加密
- 指针占几个字节
- 自己的优势和劣势
- 对开发流程的理解
- 平时写博客吗，喜欢看哪些技术文章
- 为什么离职找工作
- 切饼问题：1刀切2块，2刀切4块，10刀最多切几块。
  - 规律是在上次切得的块数上加上当前的刀数
- 追击问题：2辆火车相向同时出发，一个从A点出发，速度是30km/h,一个从B点出发，速度是20km/h，A、B之间的距离是S，同时一只鸟也从A点出发，速度是50km/h，鸟在两辆火车间折返飞行，问三者相遇时，鸟飞的总路程。

### 四面
- 梳理你遇到的最严重的问题，以及如何解决的
- 在白板上写出对于View的绘制和优化流程
- 算法：给定一段已排好序的数列，数列中元素有可能是重复的，找出数列中目标值的最小索引，要求不能用递归，只能用循环。
  - 二分查找的变种。
- 称重问题：10个箱子，每个箱子里都有若干砖头，其中有一个箱子里每个砖头重9克，其他的箱子里的砖头每个重10克，给你一个秤，要求只能用一次称重找出砖头重9克的箱子。
  - 第一个箱子拿一个砖头，第二个那两个砖头，第三个拿三个砖头，然后一起称，差几克第几个箱子都是9克的。

## 今日头条

### 一面
- 算法：连续子数组的最大和
- 了解哪些设计模式，手写单例设计模式
- View的位置参数有哪些，left，x，translationX的含义以及三者的关系
- View的绘制流程，onMeasure，onLayout的计算流程，MeasureSpec的几种模式
- 处理滑动的几种方式，Scroller滑动的原理
- 自定义View的流程，自定义View需要注意的问题，例如自定义View是否需要重写onLayout，onMeasure
- invalidate、postInvalidate、requestLayout的区别

### 二面
- TCP、UDP区别，拥塞控制原理，视频传输为什么用UDP，UDP丢包会产生什么问题，如何处理数据包的发送速度和带宽的关系
- Touch事件的分发流程
- 类的加载机制，为什么用双亲委派模型，java是否支持多态
- 插件化Activity生命周期的管理，资源的访问原理
- 算法：Given two words word1 and word2, find the minimum number of operations required to convert word1 to word2.
You have the following 3 operations permitted on a word:
1、Insert a character
2、Delete a character
3、Replace a character
Input: word1 = "horse", word2 = "ros"
Output: 3
Explanation:
horse -> rorse (replace 'h' with 'r')
rorse -> rose (remove 'r')
rose -> ros (remove 'e'

### 三面
- 算法：int[] insert(int[] arr,int len,int cap,int pos,int value)，给定一个数组，例如{1，2，3，空，空},该数组的len=3，容积cap=5，插入位置pos是1，插入的值是5，结果是{1，5，2，3，空}，返回插入后的数组（考虑扩容）；
- App的启动优化流程
- 实现一个库，完成日志的实时上报和延迟上报两种功能，该从哪些方面考虑
- 是否使用过Flutter开源框架

## 阿里搜索事业部

### 一面
- View的性能优化，listView的卡顿优化，资讯流懒加载实现。
- App的启动流程优化
- 组件化开发流程，ARouter路由协议的原理分析
- 插件化优势，插件化加载Activity原理，加载资源原理
- 插件化首次启动耗时长，如何优化
- 如何处理几百人的团队分工
- 你对淘宝App的使用体验，从哪些方面优化淘宝App

## 招商基金

### 一面
- Handler工作原理
- View的绘制原理
- 自定义View的流程
- 如何优化WebView，WebView的泄露
- native和js交互流程

### 高频考点
- 多线程知识（锁、线程池）
- 内存模型
- HashMap
- 线程间通信
- 多进程
- View绘制
- 触摸事件传递
- 链表翻转
- 快速排序
- 二分查找
- 栈相关
- int翻转
- 数组合并
- LRUCache
- MVP
- 单例
- HTTPS等等

### 知识点详细清单

- Activity的生命周期
- Activity的任务栈
- Activity的启动模式
- Fragment的生命周期
- Fragment的通讯，Fragment之间，Fragment和Activity
- 什么是Service，和Thread的区别
- Broadcast的作用和注册方式
- 什么是本地广播
- 什么是有序广播
- Android的异步处理方式有哪些
- AsycnTask、HandlerThread、IntentService源码
- Binder
- View的绘制流程
- 事件分发机制
- 自定义View的几种场景和方式
- ListView的缓存机制
- Handler、Message、MessageQueue、Looper
- 第三方开源框架设计和原理
- ANR是什么，怎么避免和排查
- OOM是什么，一般如果避免和解决
- 内存泄露是什么，常见的内存泄露有哪些
- 版本管理工具的使用，Git、SVN
- 代码编译工具
- 代码混淆
- Java IO
- 多线程
- 类加载器
- 反射
- 23种设计模式
- HTTP、TCP、UDP协议
- 计算机网络
- 操作系统原理
- 算法和数据结构：排序、二叉树遍历、动态规划
- 常见加密方式和原理

## 百度（校招）

### 一面
- 自我介绍，问了问实习期间做了什么内容，聊了聊简历上写的项目
- 了解Http吗，说说TCP三次握手四次挥手
- 为什么有三次握手，不是四次，或者两次
- Http头部有哪些，用过哪些
- Java看过源码吗，说说Collection的常用子接口和实现类，说说ArrayList和LinkedList区别
- AysncTask用过吗，各个函数分别在什么线程回调的
- 在子线程里怎么操作UI更新
- 介绍一下Handler工作原理
- 除了主线程之外其他的都要调用Looper.perpare()吗
- 用过Fragment吗，说一下Fragment和Activity的生命周期之间的关系
- 说一下Service的生命周期，用过IntentService吗有什么区别
- 如何判断单链表有环，如何判断环的位置
- 手写二分查找
- 手写伪代码：给两个栈，要求实现队列

## 阿里（校招）

### 一面
- 聊简历，问了很多围绕简历展开的问题
- 对Java的常用容器熟悉吗？知道Vector吗？说一下Hashmap和HashTable的区别
- 解释一下线程和进程之间的关系，做过多线程吗？在什么场景下应用
- 怎么保证线程同步
- Synchronize关键字的作用范围，volatile关键字的作用
- 了解死锁吗？
- 回到HashTable上，HashTable怎么保证线程同步，有没有更高效的可以保证线程同步的map，用过吗
- Hashmap看过源码吗？了解底层实现吗？Hashmap怎么解决哈希冲突的问题？对Java8的改动有关注吗？了解Java8中对Hashmap有什么优化吗
- TCP三次握手过程？是哪一层协议？说一下OSI七层模型
- 说一下TCP和UDP的区别，知道分别有什么应用场景吗
- 在无序数组中找到两个值x，y使x+y=z（z是已知值）（我给出的方法是基于排序的）。有方法可以不排序吗？
- Activity的生命周期了解吗？
- 对Activity的四种启动模式了解吗？
- 了解Android多进程开发吗？有什么应用？做过多进程开发吗？
- 进程之间有哪些通讯方式
- 说一下Binder
- ListView怎么做优化
- 自定义View有哪些关键函数？分别有什么用？自己开发过程中用过自定义View吗
- 有什么想要了解的吗

### 二面
- 看过Java什么源码
- 说一说你看过的内容
- 了解JVM的类加载吗
- 垃圾回收算法了解吗
- Java中的软引用，弱引用……
- 最近在看什么书
- 说说你了解的设计模式（开始疯狂聊设计模式，但是我是真不懂，只能想到什么说什么）

### 三面
- Activity的生命周期，需要保存当前视图的状态应该怎么做
- Activity界面绘制的过程
- Looper，Message和MessageQueue关系
- Android四大组件，都用过吗？
- 数据持久化怎么做
- Touch事件分发机制
- 讲一讲AIDL

## 美团

### 一面
- 手写代码：大整数加法
- 简历上提到使用Butterknife，Butterknife是什么阶段注解？注解的生命周期了解吗？
- 简历中提到使用Okhttp做网络通讯，说一下Okhttp拦截器
- Http和Https区别，没有CA证书可以使用Https访问吗
- TCP在是哪一层的协议，三次握手过程
- 了解什么加密算法，说一说对称加密和非对称加密
- 了解JVM吗，说说垃圾回收机制
- 了解apk的编译过程吗，简单说了一下dex
- Java中Collection的常用实现类，map的实现类
- 聊聊Hashmap和Hashtable区别
- Hashmap的底层实现方式，如何解决哈希冲突，除了这种解决方式还有没有其他解决方式
- 很多网站像新浪微博网址都有短地址，如果让你实现怎么做
- 如何判断两个单链表相交
- 如何使用两个或多个栈模拟队列
- 智力题：现在有一个抽奖活动，从8点到10点这段时间会有若干个用户参与抽奖，现在有10个获奖名额，但没有足够大的空间保存所有的数据，要求每个用户等概率中奖，且10点活动截止瞬间开奖。

### 二面
- 手写代码：删除单链表倒数第x个结点
- 手写代码：z已知，在无序数组中找到两个数x和y，使x+y=z（使用哈希表以空间换时间）
- 有没有看过Handler的源码，当MessageQueue中没有Message会怎么处理
- 说一下线程与进程，android跨进程通讯的方式，自己有没有用过
- 讲一讲AIDL原理
- Service有几种启动方式，有什么区别
- activity的声明周期和启动模式，举了一个实际场景问点击back之后的跳转
- activityA跳转到B过程中A，B的生命周期函数调用顺序
- 自定义View的相关知识
- View的事件分发机制
- 简历中提到了用Picasso做图片下载缓存，为什用Picasso，如果自己实现需要注意什么
- 说说图片三级缓存
- 知道gradle，如果module没有gradle，那能编译出来apk吗
- 有没有看过什么开源库的源码，简单介绍下
- git rebase命令是做什么的

### 三面
- 聊简历，疯狂聊简历，从头到尾
- 说说在实验室做的项目，项目中有什么难点
- 实习过程中做了什么东西，难点和亮点
- 我在github上挂了一个设备信息采集的代码，面试官看了代码和我讨论为什么这么写
- 平时看过源码吗
- 在学校社团工作中的工作内容
- 将来的职业规划，平时有什么爱好啊
- 聊了聊给我内推的学长
- 问我有没有什么觉得自己特别突出擅长的地方
- 有没有什么想要问的，想了解的

## 东方财富

### 一面
- Handler机制
- 事件分发机制
- 乐观锁与悲观锁
### 二面
- Android 7.0 8.0 p 兼容性问题
- 嵌套滑动
- 营销工具比如列表第三位展示广告，如何设计接口

## 蚂蚁金服
### 一面
- 内存泄漏
- 具体场景
- 大图加载
- 大图加载的缓存
- Bitmap优化
- Handler机制
- 子线程能不能创建Handler
- 线程间通信其他方式
- 线程的创建与退出
- 乐观锁与悲观锁
- volatile原理
- 读写锁的应用
  PublishSubject
- RecyclerView与ListView的区别
- RecyclerView为什么这么设计
- 应用
- 自定义控件
- 事件分发机制
- 动画
- okhttp支不支持优先级
- ssl握手谁实现的
- websocket应用
- 简述日志系统

### 二面
- 介绍项目架构等，围绕项目进行提问
- 序列化的作用
- 子线程轮询阻塞队列如何安全回收线程

## Musical.ly(头条)
### 一面
- 算法：数组中的数据前半部分递增，后半部分递减，排序并去重
- Java虚拟机
  - 虚拟机内存结构
  - 哪些是线程私有，哪些线程公有
  - 静态常量的存储位置
  - 无穷递归方法引发的问题
- GC机制
  - GC roots是什么
  
  - Handler造成内存泄漏的整个引用链
  Message对象有个target字段，该字段是Handler类型，引用了当前Handler对象。一句话就是：你通过Handler发往消息队列的Message对象持有了Handler对象的引用。假如Message对象一直在消息队列中未被处理释放掉，你的Handler对象就不会被释放，进而你的Activity也不会被释放。
  这种现象很常见，当消息队列中含有大量的Message等待处理，你发的Message需要等几秒才能被处理，而此时你关闭Activity，就会引起内存泄露。如果你经常send一些delay的消息，即使消息队列不繁忙，在delay到达之前关闭Activity也会造成内存泄露。
  - 什么情况会产生ANR
- 广播
  - onReceive方法调用线程
  - 静态广播接收流程
  - 动态广播接收流程
  - 动态广播能不能重复注册
- SurfaceView原理及使用注意事项
- ButterKnife工作原理
- 仿微信朋友圈图片展示设计思路
- 热修复
  - 热修复原理
  - 假如某个类A有个bug，热修复的整个流程
  - 该修复方案是否能避免oat导致的一些问题
- LeakCanary原理
- Groovy插件
  - APK瘦身如何实现的
  - 自定义任务在某个任务之前或之后执行怎么写
  - Gradle打包的整个过程
- 看过哪些Android源码
  - 拦截Activity跳转有哪些Hook点
- Groovy和Java的比较

### 二面
- 类加载过程
  - 触发类初始化的时机
  - 被动引用
- Activity启动模式及几个模式的应用场景
- onSavedInstanceState相关
  - 灭屏会不会触发onSavedInstance
  - onRestoreInstanceState和onSavedInstanceState是否成对出现
- Service生命周期的理解
  - bindService整个代码怎么写
  - 与service通信是否会阻塞当前线程
  - 如果是耗时方法，为什么会阻塞
  - 如果不是耗时方法，为什么不会阻塞
  - 如果远端是耗时操作，怎么不等待结果让主线程先运行
  - startService和bindSerivce对service生命周期的影响
  - aidl传递Bitmap需要注意的事项


## 携程
### 一面

- EventBus原理
- Java中有哪几种注解
  - 具体注解名称
  - 如何自定义注解
- EventBus是什么注解
  - 能不能用编译时注解实现EventBus
  - 注解处理器怎么工作
  - 注解处理器有哪些API
- Glide原理
- Lrucache原理
- LinkedHashpMap原理
- HashMap原理
  - 解决Hash冲突的方法
  - equals和hashcode作用
  - hashcode如何实现
- Object类下有什么方法
- 使用过哪些热修复
  - 热启动热修复原理
  - 冷启动热修复原理
- Android中的类加载器
  - 类加载器之间的区别
  - Dex融合用的哪种类加载器
  - 父类是什么及三者之间的关系
- 双亲委派模型
- APK瘦身
- Android中的动画及区别
- Handler原理
- Android中序列化方式
  - 两者区别
  - 为什么Parcelable性能更好
  - 序列化UID作用
- ThreadLocal原理
- Java中有哪些锁
  - 悲观锁与乐观锁的区别
  - 自旋锁的作用
  - 锁一般是怎么实现的
  - 让你自己实现，怎么实现一个锁
- 内存优化有哪几种方式
- 布局优化有哪几种方式
- 线性布局在onLayout里面做了哪些工作
- 谷歌为什么给Activity设计这么多生命周期

## 招行信用卡中心
- 工程结构纵向横向如何拆分（项目架构）
  - 如果解决代码依赖
  - 两个隔离模块的数据交互
- 开发过程中遇到的问题
  - 内存泄漏
- 项目中首页的实现
  - VLayout实现原理
  - 为什么不用RecyclerView实现多Item
- 热修复差分文件的验证(安全性)
- 公钥私钥体系
  - 应用场景
  - HTTPS连接过程
- 应用构建过程
- 应用签名校验过程
- V1签名和V2签名区别
- Dex加固原理
- APK瘦身

- Android
  - 四大组件、Fragment、Context
    - 概述、生命周期、启动
    - 通信方式以及一些细节，比如 Fragment 的 commitAllowingStateLoss等
  - 进程间通信
    - Binder
    - Binder 结合 AMS、WMS 等系统服务
    - 四大组件的启动过程
    - Window 的理解
  - View
    - 事件体系、滑动冲突
    - 测量布局绘制流程
    - ListView、RecyclerView 对比和缓存机制
    - invalidate、requestLayout 等刷新方式
    - SurfaceView 的原理
- 图形显示原理
  - Choreographer 结合 View 刷新
  - VSYNC、双缓冲、三缓冲
  - SurfaceFlinger
- 消息机制
- Handler、Looper、MessageQueue
- ThreadLocal
- 线程形态
  - AsyncTask
  - HandlerThread
  - IntentService
- 内存泄漏
  - 情况
  - 分析
- 性能优化
  - 常见的一些套路，比如布局优化、内存泄漏优化、ListView/RecyclerView 优化、LruCache 等
  - 可以结合图形显示原理和Handler机制扩展
  - 大图显示
- 开源库，这部分通常会考察源码的设计和实现
  - EventBus
  - Retrofit
  - OkHttp
  - 其他项目中用到的流行开源库
- 其他
  - Android 版本变更
  - APK 打包、安装过程
  - Android 类加载，结合 Java 类加载
  - Dalvik/ART，结合 Java 虚拟机
  - 进程保活、进程优先级
  - JNI，这块不用掌握很深，大致原理知道就行
  - SharePreferences 的原理和注意点
  - Parcelable 与 Serializable
  - WebView
  - MultiDex
- Java
  - 语言
    - 动态代理
    - 类型信息、反射
    - 泛型
    - 异常
- 容器，HashMap一定要看，其他的各种常见容器也都建议了解一下实现
  - 常用容器：HashMap、LinkedHashMap、ArrayList、LinkedList等
  - Android 容器：SparseArray 等
  - 并发容器：ConcurrentHashMap、CopyOnWriteArrayList、阻塞队列
- 并发
  - 线程状态
  - 内置/显式的锁、条件队列
  - 死锁
- 线程池
  - volatile、原子变量、CAS、ABA
  - 内存模型和 happens-before
- 虚拟机
  - 内存区域
  - 对象内存分配
  - 引用计数和可达性分析
  - 垃圾收集算法和垃圾收集器
- 设计模式
  - 常用设计模式，常考单例、模板、观察者
  - 源码中的设计模式
  - MVC/MVP/MVVM
- 计算机基础
  - 计算机网络
    - HTTP：请求、响应、缓存、版本变化
    - TCP/IP：握手挥手、流量控制、拥塞控制、广播/组播等。
  - 操作系统
    - 进程和线程
    - 虚拟内存、分页和分段
  - 数据库
    - 索引
    - 事务和ACID
    - 关系型和非关系型
    - ORM和SQL

## 阿里
### 一面
- 自我介绍
- 项目来历
- MySQL 和 MongoDB 的区别
- 关系型数据库和非关系型数据库的区别，各自在什么情况下使用
- 为什么海量数据时适合用非关系型数据库
- 为什么研究生读的通信，而不是计算机
- 上过或自学过哪些计算机课程，对计算机网络挺熟悉的吧
- 对 HTTPS 有没有了解，有没有写过相关代码
- 老板有 2000 元给我和同学分钱，首先由我提出分钱方案。如果同学不同意，则总额变为 1000 元，并由同学提出分钱方案。如果这时我不同意，则俩人各拿 100 元。请问一开始我应该怎么给出分钱方案？
- 假设有一个线程在取队列中的消息，怎么停止这个线程
- 读过哪些源码，分别说一下 Android 和 Java 的
- 读过哪些 Android 开源库源码
- 项目中遇到过什么难题
- 印象深刻的几次学习经历
- 看你用过 Ubuntu，平时是不是用 Linux，熟悉Linux吗，都用它干什么
- 有什么问题想问的

### 二面
- 介绍一下项目
- 一连串问了多个问题关于项目中给设备配置入网过程的细节
- 项目中与设备通信的数据格式，还可以用哪些数据格式，做过什么优化
- 设计一种变长的传输数据格式
- UDP 和 TCP
- 介绍一下 MQTT 协议（项目中用到的一个协议）
- 项目中有没有做过一些监控日志
- Elasticsearch是什么（因为我说到才问的）
- CrashHandler 的实现
- 设计一个日志监控和上传模块，深入问了很多细节和考虑的方面
- Android 各个版本的变化
- 介绍一下 Flutter 框架
- 平时怎么了解和学习新知识的

### 三面
- 问了很多项目细节，关于本地存储、音乐播放等基础模块的封装
- 有没有把音乐播放模块开源出去
- 项目中解决过什么难题
- HTTP/2.0 有什么变化
- 在一个手机屏幕上有两列（类似于瀑布流），给你一些高度不同的 item，怎么摆放才能让这两列的整体高度最小？

## 头条

### 一面
- 自我介绍
- 项目来历
- 项目中本地存储怎么做的
- 访问本地存储的类应该怎么设计
- 说一下对 BInder 的理解
- Binder 红黑树的节点是以什么区分的
- SurfaceFlinger、VSYNC
- SurfaceView 原理
- HTTP 响应内容
- Retrofit 内部实现
- 动态代理的原理
- EventBus 内部实现
- 手写二分查找

### 二面
- 看过哪些书
- 说一下《Java 并发编程实战》的作者有哪些人，还认识哪些大牛，比较喜欢谁，会去关注吗
- HashMap 版本变化和原理
- 手写快速排序并解释
- 各种排序的复杂度和稳定性
- 类成员的权限怎么定

### 三面
- JVM 内存区域
- volatile 和重排序
- Java 内存模型
- 引用计数和可达性分析
- finalize 原理
- Android 界面刷新原理
- Android 性能优化有哪些方法
- 观察者模式、责任链模式
- OkHttp 内部实现、有没有缓存
- 从输入一个 URL 到看到一个页面的过程
- 看过哪些书
- 是不是实验室做开发的同学中最强的

## 腾讯
### 一面
- 能不能来深圳
- 介绍项目
- 为什么用 SharedPreferences，怎么存 JSON
- 项目中遇到什么难题
- 广播和组播
- 重男轻女，生女孩会生到男孩为止，问男女比例
- Gson原理
- SQL语句、数据库优化
- ORM、DAO、DSL
- 模板方法模式
### 二面
- 上过哪些计算机课程
- Java 类加载
- Android 类加载，DexClassLoader有什么用
- 散列表有哪些解决冲突的方式
- LinkedList 和 ArrayList 比较
- TCP 的拥塞控制
- 能不能来深圳


## 美团
### 一面
- 自我介绍
- 项目来历
- 为什么喜欢 Java，然后问了一些 Java 的特性
- 线程和协程，为什么协程比线程效率高
- RecyclerView 图片错位问题
- Fragment 的 commitAllowStateLoss 方法
- Activity 与 Fragment 怎么通信
- Handler 怎么解决内存泄漏问题
- 其他内存泄漏情况
- 单击事件和双击事件哪个先触发
- selector 为什么能够切换背景，原理是什么
- 不考虑具体页面，怎么从根本上优化界面卡顿
- SurfaceFlinger、VSYNC
- 描述一下 Handler 的原理
- 依次打印二叉树每层最左边的结点

### 二面
- 怎么判断对象是否要进入老年代
- Eden 和 Survivor 的比例和回收规则
- 新生代为什么用复制算法
- 说一下对 Binder 的理解
- 说一下 ActivityManagerService、ActivityManagerNative 等几个类的区别
- 手写各种单例模式
- 跳台阶问题
- 求两个链表的交点
- 判断二叉树是否左右对称（只考虑结构对称，不考虑值）

### 三面
- MVP 及 MVP 怎么解决内存泄漏
- 项目架构
- 说一下 SharedPreferences
- 给一个数组，判断是否存在一对相等的前缀和与后缀和
- 平时怎么学习的
- 有什么想学的新技术、学习计划

