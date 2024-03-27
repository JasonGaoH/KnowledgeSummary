
### 注解基础
元注解 @Retention：
表示注解存在于什么阶段，有以下三种，
RetentionPolicy.SOURCE: 只存在于源码，编译期就会被擦除。类似于@override，只是给开发人员看的。
RetentionPolicy.CLASS: 存在于源码和编译期，不在运行时。是默认的policy。
RetentionPolicy.RUNTIME: 存在于源码、编译期、运行时。

https://www.race604.com/annotation-processing/


### 调试注解处理器代码
http://www.jianshu.com/p/80a14bc35000

1）在自定义AbstractProcessor#process()添加断点

2）project/ gradle.properties添加如下配置，并sync。如果端口被占用，可以换个端口
org.gradle.daemon=true
org.gradle.jvmargs=-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=5006

3）命令行运行 ./gradlew --daemon，启动守护线程

4）在AS中新建Remote Debugger (如果有，只需要在Select Configuration中选中AnnotationProcessor对应的Remote Debugger)，然后点击Debug按钮，启动调试。注意端口要和gradle.properties配置的端口一致。




5）命令行运行 ./gradlew clean connectedCheck
注意，如果执行改命令仍然不能调试，那就./gradlew --stop关闭所以daemon，再./gradlew --deamon重启，即3，4，5步重新走一遍。

常见问题：
1）gradle.properties添加daemon配置同步后，AS中出现如下

A: daemon默认占用5005端口，上面错误说明该端口已经被占用，可以修改成其他端口。

常见命令：
./gradlew --daemon 启动守护线程
./gradlew --daemon --stop 关闭所以守护线程
./gradlew --daemon --status 查看守护线程状态

### 注解处理器精髓
工作大体内容：
1) 获取被注解的Element;
2) 代码校验，比如被注解的类要能实例化，所以要是public，不能是abstract，方法必须public等；
3) 根据Element获取要生成代码的信息，结合JavaPoet，生成代码；

JavaPoet：https://github.com/square/javapoet

======================== Element =========================
Element:


TypeElement:
     代表的是源代码中的类型元素，例如类。然而，TypeElement并不包含类本身的信息。你可以从TypeElement中获取类的名字，但是你获取不到类的信息，例如它的父类。这种信息需要通过TypeMirror获取。你可以通过调用elements.asType()获取元素的TypeMirror。


======================== 创建Compiler Module =========================
创建一个名为compiler的Java Library，这个类将会写代码生成的相关代码。核心就是在这里。

配置build.gradle

apply plugin: 'java'
sourceCompatibility = 1.7
targetCompatibility = 1.7
dependencies {
compile fileTree(dir: 'libs', include: ['*.jar'])
compile 'com.google.auto.service:auto-service:1.0-rc2'
compile 'com.squareup:javapoet:1.7.0'
compile project(':annotation')
}

定义编译的jdk版本为1.7，这个很重要，不写会报错。
AutoService 主要的作用是注解 processor 类，并对其生成 META-INF 的配置信息。
JavaPoet 这个库的主要作用就是帮助我们通过类调用的形式来生成代码。
依赖上面创建的annotation Module。
