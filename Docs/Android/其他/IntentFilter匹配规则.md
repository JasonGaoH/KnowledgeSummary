# IntentFilter匹配规则

Activity隐式调用时，需要Intent能够匹配目标组件的IntentFilter设置的过滤信息。

IntentFilter设置的过滤信息有action、category、data。

### action匹配规则

1. Intent中必须有且一个action，没有action会匹配异常
2. Intent中的action，必须和目标组件IntentFilter中多个action中一个严格匹配（区分大小写），才能算action匹配成功。

注意：Intent中只能设置一个action，Intent.setAction()

目标组件：

```
 <activity
            android:name="com.facebook.CustomTabActivity"
            android:exported="true">
            <intent-filter>
                <action android:name="com.shannon.Activity1" />
                <action android:name="com.shannon.Activity2" />

                <category android:name="android.intent.category.DEFAULT" />
                <category android:name="android.intent.category.BROWSABLE" />

                <data android:scheme="@string/fb_login_protocol_scheme" />
            </intent-filter>
        </activity>

```

所以，

```
Intent.setAction("com.shannon.Activity1")  //匹配通过

Intent.setAction("com.shannon.Activity2")  //匹配通过

Intent.setAction("com.shannon.Activity3")  //匹配失败，目标组件没有Activity3这个action
```

### category匹配规则

Intent中可以添加多个category。要求这些category中的每一个，都必须被包含在目标组件IntentFilter中的category里。


```
Intent.addCategory("android.intent.category.DEFAULT")
Intent.addCategory("android.intent.category.BROWSABLE")
匹配通过

Intent.addCategory("android.intent.category.DEFAULT") //目标组件能匹配这个
Intent.addCategory("android.intent.category.OTHER") //目标组件能无法匹配这个
不能全部匹配成功，所以结果是：匹配失败
```

### data匹配规则

data匹配规则和action类似，Intent的data必须和目标组件IntentFilter多个data中的一个严格匹配，才算匹配成功。

data 结构：scheme://host:port/path...

结构解释：

* Scheme：比如file、content、http等，如果URI中没指定scheme，那整个URI其他参数无效，也意味着URI是无效的。
* Host：如果host未指定，整个URI其他参数无效，意味着URI也是无效的。


如果没指定scheme，系统会默认加上content和file。如下所示：

```
<intent-filter>
    <data android:mimeType="image/*" />
</intent-filter>

使用Intent.setDataAndType(Uri.parse("file://abc"), "image/png")，能匹配
使用Intent.setDataAndType(Uri.parse("http://abc"), "image/png")，无法匹配

```


```
<intent-filter>
    <action android:name="com.shannon.demo.jump" />
    <data
        android:host="shannon"
        android:scheme="router" />
</intent-filter>

以下Intent可以匹配上述目标组件：
Intent intent = new Intent("com.shannon.demo.jump");
intent.setData(Uri.parse("shannon://router"));
```






