# launchMode

Activity四种启动模式。

##### standard：标准模式

每次启动都会创建一个新的实例。

该模式下，新启动的Activity会放在启动它的Activity所在的栈中。所以使用Application Context启动Activity会报错，因为非Activity的Context并没有所谓的任务栈。解决此问题的方法是待启动Activity指定``FLAG_ACTIVITY_NEW_TASK``标记位，这样，会先新建一个任务栈，再把待启动的Activity放到新的栈中。相当于singleTask模式。

##### singleTop: 栈顶复用

只有待启动Activity在栈顶时，才会复用。

即待启动的Activity位于栈顶时，这种启动方式下不会创建新的，而是复用栈顶的那个Activity，并且调用它的onNewIntent()。如果待启动Activity不在栈顶，就创建个新的放在栈顶。
    示例，ABCD情况下，A在栈底，D在栈顶。1）启动D，结果是ABCD；2）启动B，结果是ABCDB。

##### singleTask: 栈内复用

同一个栈内，如果有待启动的Activity，则复用该Activity。

复用时，和singleTop一样，回调onNewIntent()。

该模式具有clearTop作用，即待启动Activity上调到栈顶时，位于它前面的Activity会被清除。
    具体解释，具有singleTask模式的Activity A被启动时，系统会首先找是否存在A想要的任务栈，如果没有，就新建个任务栈，并创建A的实例放到栈中。如果存在A需要的任务栈，就看栈中有没有A的实例。如果有，把A调到栈顶，并调用它的onNewIntent()，如果没有，就新建A的实例压入栈中。
    示例，ABCD情况下，A在栈底，D在栈顶。启动B，结果是AB。

##### singleInstance: 单例模式

和singleTask一样，但是加强了一点，就是该模式下的Activity会独占一个栈。
    
    
**需要注意的一点：**

必须用Activity Context启动Activity，如果用Application Context会crash。

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

