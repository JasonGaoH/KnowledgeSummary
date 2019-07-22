## Java类加载流程
三个类类加载器：
 - Bootstrap ClassLoader 最顶层的加载类，主要加载核心类库，%JRE_HOME%\lib下的rt.jar、resources.jar、charsets.jar和class等
 - Extention ClassLoader 扩展的类加载器，加载目录%JRE_HOME%\lib\ext目录下的jar包和class文件
 - Appclass Loader也称为SystemAppClass 加载当前应用的classpath的所有类

 
BootstrapClassLoader加载的jar包路径
```
System.out.println(System.getProperty("sun.boot.class.path"));
```


```
C:\Program Files\Java\jre1.8.0_91\lib\resources.jar;
C:\Program Files\Java\jre1.8.0_91\lib\rt.jar;
C:\Program Files\Java\jre1.8.0_91\lib\sunrsasign.jar;
C:\Program Files\Java\jre1.8.0_91\lib\jsse.jar;
C:\Program Files\Java\jre1.8.0_91\lib\jce.jar;
C:\Program Files\Java\jre1.8.0_91\lib\charsets.jar;
C:\Program Files\Java\jre1.8.0_91\lib\jfr.jar;
C:\Program Files\Java\jre1.8.0_91\classes
```

ExtClassLoader的加载路径

```
System.out.println(System.getProperty("java.ext.dirs"));
```

```
C:\Program Files\Java\jre1.8.0_91\lib\ext;C:\Windows\Sun\Java\lib\ext
```


### 每个类加载器都有一个父加载器
### 父加载器不是父类

### Bootstrap ClassLoader是由C++编写的

### 双亲委托
 
> 一个类加载器查找class和resource时，是通过“委托模式”进行的，它首先判断这个class是不是已经加载成功，如果没有的话它并不是自己进行查找，而是先通过父加载器，然后递归下去，直到Bootstrap ClassLoader，如果Bootstrap classloader找到了，直接返回，如果没有找到，则一级一级返回，最后到达自身去查找这些对象。这种机制就叫做双亲委托。

[link](https://blog.csdn.net/briblue/article/details/54973413)

为什么采用双亲委托机制？
[link](https://crossoverjie.top/JCSprout/#/jvm/ClassLoad)

> 双亲委派的好处 : 由于每个类加载都会经过最顶层的启动类加载器，比如 java.lang.Object这样的类在各个类加载器下都是同一个类(只有当两个类是由同一个类加载器加载的才有意义，这两个类才相等。)

如果没有双亲委派模型，由各个类加载器自行加载的话。当用户自己编写了一个 java.lang.Object类，那样系统中就会出现多个 Object，这样 Java 程序中最基本的行为都无法保证，程序会变的非常混乱。

-----------------------------------
jvm如何认定两个对象同属于一个类型，必须同时满足下面两个条件：

A.都是用同名的类完成实例化的。

B.两个实例各自对应的同名的类的加载器必须是同一个。比如两个相同名字的类，一个是用系统加载器加载的，一个扩展类加载器加载的，两个类生成的对象将被jvm认定为不同类型的对象。

所以，为了系统类的安全，类似“java.lang.Object”这种核心类，jvm需要保证他们生成的对象都会被认定为同一种类型。即“通过代理模式，对于 Java 核心库的类的加载工作由引导类加载器来统一完成，保证了 Java 应用所使用的都是同一个版本的 Java 核心库的类，是互相兼容的”。
