

### 方案对比

当前比较流行的热补丁方案有微信Tinker、阿里AndFix、美团Robust、QZone超级补丁。每个平台都有自己的局限，大家在技术选型时可以根据业务需要做选择。

| 特性 | Tinker |	QZone |	AndFix | Robust |
| ------|:-------|:------:| -----:| -----:|
| 类替换 | yes | yes | no | no | 
| So替换	| yes	| no	| no| 	no| 
| 资源替换| 	yes| 	yes| 	no| 	no| 
| 全平台支持| 	yes| 	yes| 	yes| yes| 
| 即时生效| 	no| 	no| 	yes| 	yes| 
| 性能损耗| 	较小| 	较大| 	较小| 	较小| 
| 补丁包大小| 	较小| 	较大| 	一般| 一般| 
| 开发透明| 	yes| 	yes| 	no| 	no| 
| 复杂度| 	较低| 	较低| 	复杂| 	复杂| 
| gradle支持| 	yes| 	no| 	no| no| 
| Rom体积| 	较大| 	较小| 	较小| 	较小| 
| 成功率| 	较高| 	较高| 	一般| 	最高| 


总的来说:

1. AndFix作为native解决方案，首先面临的是稳定性与兼容性问题，更重要的是它无法实现类替换，它是需要大量额外的开发成本的；
2. Robust兼容性与成功率较高，但是它与AndFix一样，无法新增变量与类只能用做的bugFix方案；
3. Qzone方案可以做到发布产品功能，但是它主要问题是插桩带来Dalvik的性能问题，以及为了解决Art下内存地址问题而导致补丁包急速增大的。


### Tinker已知问题

由于原理与系统限制，Tinker有以下已知问题：

1. Tinker不支持修改AndroidManifest.xml，Tinker不支持新增四大组件(1.9.0支持新增非export的Activity)；
2. 由于Google Play的开发者条款限制，不建议在GP渠道动态更新代码；
3. 在Android N上，补丁对应用启动时间有轻微的影响；
不支持部分三星android-21机型，加载补丁时会主动抛出"TinkerRuntimeException:checkDexInstall failed"；
对于资源替换，不支持修改remoteView。例如transition动画，notification icon以及桌面图标。

### 技术解析

##### 4. 如何打差分包，即补丁文件

使用DexDiff算法，[参考](https://www.zybuluo.com/dodola/note/554061)


##### 5. 不同API level，哪些环节需要分别处理

加载dex文件会区分API level。<br>

Level 24之前：<br>
采用类似于MultiDex加载dex文件的原理，将补丁dex在dexElements中前置。因为谁在前面用谁，所以系统会使用补丁代码，从而达到热修复功能。

Level 24及其之后：<br>


```
补充知识：
API Level 26 : Android Oreo, 8.0
API Level 24 : Android Nougat, 7.0
API Level 23 : Android Marshmallow, 6.0
API Level 21 : Android Lollipop, 5.0
API Level 19 : Android Kitkat, 4.4
```

##### 6. MultiDex原理？Tinker参考了MultiDex哪些技术？

##### 7. InstantRun原理？Tinker参考了InstantRun哪些技术？

更新资源文件时，借鉴了InstantRun的技术。




#### 重要点
1. Tinker的补丁方案，Tinker采用的是下发差分包，然后在手机端合成全量的dex文件进行加载。

##### 加载补丁dex文件
使用PathClassLoader加载补丁dex。将补丁dex和原dex合并成fix_class.dex，用这个新的dex替换原有pathDexList中的内容。

##### 加载补丁so文件

##### 加载补丁资源文件
采用InstantRun的加载补丁资源方式，全量替换资源。实现方式是hook AssetManager.addAssetPath()，将补丁的资源目录传进去，以此达到替换老资源的方式。

InstantRun的资源更新方式最简便而且兼容性也最好，市面上大多数的热补丁框架都采用这套方案。Tinker的这套方案虽然也采用全量的替换，但是在下发patch中依然采用差量资源的方式获取差分包，下发到手机后再合成全量的资源文件，有效的控制了补丁文件的大小。


### 类加载原理

Android中类的加载也是通过ClassLoader来完成，具体来说就是PathClassLoader和DexClassLoader 这两个Android专用的类加载器，这两个类的区别如下：

* PathClassLoader：只能加载已经安装到Android系统中的apk文件（/data/app目录），是Android默认使用的类加载器。
* DexClassLoader：可以加载任意目录下的dex/jar/apk/zip文件，也就是我们一开始提到的补丁。
这两个类都是继承自BaseDexClassLoader，我们可以看一下BaseDexClassLoader的构造函数。

```
public BaseDexClassLoader(String dexPath, File optimizedDirectory,
            String libraryPath, ClassLoader parent) {
	super(parent);
	this.pathList = new DexPathList(this, dexPath, libraryPath,optimizedDirectory);
}
```

这个构造函数只做了一件事，就是通过传递进来的相关参数，初始化了一个DexPathList对象。DexPathList的构造函数，就是将参数中传递进来的程序文件（就是补丁文件）封装成Element对象，并将这些对象添加到一个Element的数组集合**dexElements**中去。

假设我们现在要去查找一个名为name的class，那么DexClassLoader将通过以下步骤实现：

* 在DexClassLoader的findClass 方法中通过一个DexPathList对象findClass()方法来获取class
* 在DexPathList的findClass 方法中，对之前构造好dexElements数组集合进行遍历，一旦找到类名与name相同的类时，就直接返回这个class，找不到则返回null。

总的来说，通过DexClassLoader查找一个类，最终就是就是在一个数组中查找特定值的操作。


参考文章：

[Tinker](https://github.com/Tencent/tinker/wiki)
[Android热补丁之Tinker原理解析](http://w4lle.com/2016/12/16/tinker/)

