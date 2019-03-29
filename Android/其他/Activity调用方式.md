# Activity调用方式 (显式&隐式)

Activity调用方式分为显示和隐式两种，区别是：

显示必须明确指定待启动对象的组件信息，包括包名和类名。隐式调用则不需要指明组件信息。

### 显示调用

```
方法1：
val intent = Intent(this@MainActivity, TargetActivity::class.java)
startActivity(intent)

方法2：
val intent = Intent()
intent.setClass(this@MainActivity, TargetActivity::class.java)
startActivity(intent)

方法3.
val intent = Intent()
intent.component = ComponentName(this@MainActivity, TargetActivity::class.java)
startActivity(intent)
```

其中，方法1和2，内部实现都是构建一个ComponentName，即方法3。


### 隐式调用

```
方法1. 通过action跳转，前提是目标Activity在xml中注册过对应action

val intent = Intent()
intent.setAction("jumpTargetActivityAction")
startActivity(intent)

XML：
<activity android:name=".TargetActivity" >
    <intent-filter>
        <action android:name="jumpTargetActivityAction"/>
        <category android:name="android.intent.category.DEFAULT" />
    </intent-filter>
</activity>
```
```
方法2. 通过scheme跳转（RED的逻辑）

val uri = Uri.parse("ascheme://ahost:8080/apath?user=xiaoming")
val intent = Intent(Intent.ACTION_VIEW, uri) //这种方式其实内部也是把uri放到Data里，即setData形式
startActivity(intent)

或者

val uri = Uri.parse("ascheme://ahost:8080/apath?user=xiaoming")
val intent = Intent(Intent.ACTION_VIEW)
intent.setData(uri)
startActivity(intent)

XML：
<activity android:name=".TargetActivity" >
    <intent-filter>
        <category android:name="android.intent.category.DEFAULT"/>
        <action android:name="android.intent.action.VIEW"/>
        <category android:name="android.intent.category.BROWSABLE"/>

        <data
            android:scheme=“ascheme"
            android:host=“ahost"
            android:path=“/apath"
            android:port="8080"/>
    </intent-filter>
</activity>
```