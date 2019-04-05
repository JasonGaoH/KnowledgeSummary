# APK打包及安装过程

* APK打包过程
* APK安装过程
* APK文件结构

### APK打包过程



### APK安装过程

Android应用安装涉及到如下几个目录：

* /data/app : 存放用户安装的apk的目录，安装时，把apk拷贝于此。
* /data/data : 安装完成后，在/data/data下生成与应用包名一致的文件夹，用于存放应用的数据(file/cache等)。
* /data/dalvik-cache : 存放APK的odex文件，便于应用启动时直接执行。


安装过程如下：

* 首先，复制APK安装包到/data/app下
* 然后校验APK的签名是否正确，检查APK的结构是否正常
* 进而解压并且校验APK中的dex文件，确定dex文件没有被损坏后，再把dex优化成odex，使得应用程序启动时间加快
* 同时在/data/data目录下建立于APK包名相同的文件夹
* 如果APK中有lib库，系统会判断这些so库的名字，查看是否以lib开头，是否以.so结尾，再根据CPU的架构解压对应的so库到/data/data/packagename/lib下。


APK安装的时候会把DEX文件解压并且优化位odex，odex的格式如Figure 5图所示：



odex在原来的dex文件头添加了一些数据，在文件尾部添加了程序运行时需要的依赖库和辅助数据，使得程序运行速度更快。


### APK文件结构



参考内容：

[APK文件结构和安装过程](https://mp.weixin.qq.com/s?__biz=MzI3MDE0NzYwNA==&mid=2651433396&idx=1&sn=180d2285d5c1c61aeeed4462ae7d75a9&scene=23&srcid=0525qQiTtYdC1rllR0q0maOm#rd)