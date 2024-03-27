### Activity的生命周期

![](../../img/activity_lifecycle.jpg)

#### 问题：使用Application Context启动Activity为什么会crash

示例：

```
val intent = Intent(this@MainActivity, SecondActivity::class.java)
applicationContext.startActivity(intent)

```

Crash Log:

```
Caused by: android.util.AndroidRuntimeException: Calling startActivity() from outside of an Activity  context requires the FLAG_ACTIVITY_NEW_TASK flag. Is this really what you want?
```

原因是，被启动Activity默认放在启动它的Activity所在的栈。Application Context没有Activity栈，所以会异常。

解决方案是，使用``flag:FLAG_ACTIVITY_NEW_TASK``，让被启动Activity处在新建的栈。

```
val intent = Intent(this@MainActivity, SecondActivity::class.java)
intent.flags = Intent.FLAG_ACTIVITY_NEW_TASK
applicationContext.startActivity(intent)
```

#### 问题：类似微博分享功能适合的launchMode，为什么不是singleInstance
> 分享页面有跳转其他页面按钮，点击跳转其他页面，如果使用singleInstance，点击回退，就回不到分享页面了。这里需要注意的是，singleInstance是独占一个栈的。

无论 Dialog 弹出覆盖页面，对 Activity 生命周期没有影响，只有再启动另外一个 Activity 的时候才会进入 onPause 状态，而不是想象中的被覆盖或者不可见.

同时通过 AlertDialog 源码或者 Toast 源码我们都可以发现它们实现的原理都是 windowmanager.addView();来添加的， 它们都是一个个 view ,因此不会对 activity 的生命周期有任何影响。

https://blog.csdn.net/cloud_castle/article/details/56011562