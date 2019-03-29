# Android多进程

本节知识点：

* 开辟新进程的方法--android:process
* 使用多进程的问题
* PID与UID区别

### 开辟新进程的方法--android:process

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

### PID与UID区别

PID：
	
为Process Identifier，PID就是各进程的身份标识,程序一运行系统就会自动分配给进程一个独一无二的PID。进程中止后PID被系统回收，可能会被继续分配给新运行的程序，但是在android系统中一般不会把已经kill掉的进程ID重新分配给新的进程，新产生进程的进程号，一般比产生之前所有的进程号都要大。

UID：

一般理解为User Identifier,UID在linux中就是用户的ID，表明时哪个用户运行了这个程序，主要用于权限的管理。而在android 中又有所不同，因为android为单用户系统，这时UID 便被赋予了新的使命，数据共享，为了实现数据共享，android为每个应用几乎都分配了不同的UID，不像传统的linux，每个用户相同就为之分配相同的UID。（当然这也就表明了一个问题，android只能时单用户系统，在设计之初就被他们的工程师给阉割了多用户），使之成了数据共享的工具。


因此在android中PID和UID都是用来识别应用程序的身份的，但UID是为了不同的程序使用共享的数据。
在android 中要通过UID共享数据只需在程序a,b中的menifest配置即可，具体如下：

```
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
          package="com.perseus.a"
　　　　　 android:versionCode="1"
　　　　　 android:versionName="1.0"
          android:sharedUserId="com.share"
>

<manifest xmlns:android="http://schemas.android.com/apk/res/android"
          package="com.perseus.b"
　　　　　 android:versionCode="1"
　　　　　 android:versionName="1.0"
          android:sharedUserId="com.share"
>
```

这样我们就可以在a程序中通过跳转activity的形式访问b中的数据了。
   这样的话你也许会有疑问，如果让其他的开发这知道了我们的shareUserId知道了我们的ID，那我们的数据不是暴露了，放心吧google不会犯这样的低级错误的，我们要使不同的程序能够相互访问，还需要拥有相同的签名，每个公司或者开发者的签名是唯一的，这样我们就不用担心了，另外两者能够访问，别忘了权限

注意：两个应用通过shareUserId共享数据时，这两个应用的签名也要必须相同！

## Add
补充：之前遇到过的问题，在应用Application中写入SharedPreference时，在多进程情况下，每个进程都会走Application的创建，这样会导致进程间会有数据的同步问题，考虑在写入Sp的时候判断当前的进程。

