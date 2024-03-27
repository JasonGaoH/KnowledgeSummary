### 什么是IPC
> IPC是Inter-Process Communication的缩小，含义为进程间通信或者跨进程通信，是指两个进程之间进行数据交换的过程。

### 进程和线程
> 进程：正在进行中的程序。其实进程就是一个应用程序运行时的内存分配空间。 线程：其实就是进程中一个程序执行控制单元，一条执行路径。 进程负责的是应用程序的空间的标示。线程负责的是应用程序的执行顺序。

### Androrid中的多进程模式

Android的四大组件（Activity/Service/BroadcastReceiver/ContentProvider）注册时，如果使用android:proces修饰，说明该组件创建在新的进程中。

android:process：作用是设置进程名称。

android:process 两种声明方式：

* android:process=“:remote" 
	以：打头的进程属于当前应用的私有进程，其他应用程序的组件不可以和它跑在同一进程中。
	
* android:process=“com.utils.myapplication.user"
	以包名形式声明的进程，属于全局进程，其他应用可以通过shareUserId方式和该进程跑在同一进程中，从而分享应用间的私有数据。但是这两个应用的签名还必须相同，才能共享私有数据，比如data目录、组件信息等。

### 使用多进程的问题

1. 静态成员和单例模式失效；
2. 线程同步机制完全失效；
3. SharePreference可靠性下降；
4. Application会多次创建。

### Android中的IPC方式

#### 使用Bundle
Android中四大组件中的三大组件都是支持在Intent中传递Bundle数据的，由于Bundle事先了Parcelable接口，所以它可以方便地在不同的进程间传输。

GitHub上有个关于使用Bundle的讨论：[Android里面为什么要设计出Bundle而不是直接用Map结构](https://github.com/android-cn/android-discuss/issues/142)

#### 使用文件共享

#### 使用Messenger 

#### 使用AIDL

#### 使用ContentProvider

#### 使用Socket

#### Add
补充：之前遇到过的问题，在应用Application中写入SharedPreference时，在多进程情况下，每个进程都会走Application的创建，这样会导致进程间会有数据的同步问题，考虑在写入Sp的时候判断当前的进程。