> 文章已同步发表于微信公众号JasonGaoH，[Java基础知识总结](https://mp.weixin.qq.com/s?__biz=MzUyNTE2OTAzMQ==&mid=2247483714&idx=1&sn=d8ff6baf229c463802687e4d79751106&chksm=fa2379aacd54f0bc808dd6a929c3d571f780367a140c6fd30c5b86a9e708692003aa3d197a7b&token=1938879438&lang=zh_CN#rd)

- [1. Java 面向对象编程三大特性: 封装 继承 多态](#1-java-面向对象编程三大特性-封装-继承-多态)    
  - [封装](#封装)    
  - [继承](#继承)    
  - [多态](#多态)
- [2. Java对象的生命周期](#2-java对象的生命周期)
- [3. 抽象类和接口的区别](#3-抽象类和接口的区别)   
-  [抽象类与接口：](#抽象类与接口)
   - [常用关键字this、super、static和final](#常用关键字thissuperstatic和final)  
     - [this: 代表对象。就是所在函数所属对象的引用。](#this-代表对象就是所在函数所属对象的引用)    
     - [super关键字](#super关键字)
     - [static：关键字，是一个修饰符，用于修饰成员(成员变量和成员函数)。](#static关键字是一个修饰符用于修饰成员成员变量和成员函数)    
     - [静态代码块：就是一个有静态关键字标示的一个代码块区域。定义在类中。](#静态代码块就是一个有静态关键字标示的一个代码块区域定义在类中)    
     - [final：关键字](#final关键字)
- [4. StringBuffer和StringBuilder的区别](#4-stringbuffer和stringbuilder的区别)
- [5. Object类的equal和hashcode方法](#5-object类的equal和hashcode方法)    - [hashCode（）与equals（）的相关规定](#hashcode与equals的相关规定)
- [6. == 与 equals](#6--与-equals)
- [7. String为什么要设计成不可变的](#7-string为什么要设计成不可变的)
- [8.可否用try-catch捕获OOM以避免其发生](#8可否用try-catch捕获oom以避免其发生)
- [9. 单例设计模式](#9-单例设计模式)
- [10.内部类](#10内部类)
- [11. 多线程](#11-多线程)
- [12.为什么要有Runnable接口的出现](#12为什么要有runnable接口的出现)
- [13.线程间通信：思路：多个线程在操作同一个资源，但是操作的动作却不一样。](#13线程间通信思路多个线程在操作同一个资源但是操作的动作却不一样)    
  - [等待唤醒机制：涉及的方法：](#等待唤醒机制涉及的方法)   
  -  [wait和sleep区别： 分析这两个方法：从执行权和锁上来分析：](#wait和sleep区别-分析这两个方法从执行权和锁上来分析)
-  [14.Lock接口](#14lock接口)
-  [15. 对象的序列化](#15-对象的序列化)    
   - [transient使用细节——被transient关键字修饰的变量真的不能被序列化吗？](#transient使用细节被transient关键字修饰的变量真的不能被序列化吗)


## 1. Java 面向对象编程三大特性: 封装 继承 多态

### 封装

是指隐藏对象的属性和实现细节，仅对外提供公共访问方式。
- 好处：
  - 将变化隔离
  - 便于使用
  - 提高重用性
  - 安全性
- 封装原则：
  - 将不需要对外提供的内容都隐藏起来，把属性都隐藏，提供公共方法对其访问。

### 继承

好处：
1. 提高了代码的复用性。
2. 让类与类之间产生了关系，提供了另一个特征多态的前提。
 
父类的由来：其实是由多个类不断向上抽取共性内容而来的。
java中对于继承，java只支持单继承。java虽然不直接支持多继承，但是保留了这种多继承机制，进行改良。

- 单继承：一个类只能有一个父类。
- 多继承：一个类可以有多个父类。

为什么不支持多继承呢？
>因为当一个类同时继承两个父类时，两个父类中有相同的功能，那么子类对象调用该功能时，运行哪一个呢？因为父类中的方法中存在方法体。
但是java支持多重继承。A继承B  B继承C  C继承D。
多重继承的出现，就有了继承体系。体系中的顶层父类是通过不断向上抽取而来的。它里面定义的该体系最基本最共性内容的功能。
所以，一个体系要想被使用，直接查阅该系统中的父类的功能即可知道该体系的基本用法。那么想要使用一个体系时，需要建立对象。建议建立最子类对象，因为最子类不仅可以使用父类中的功能。还可以使用子类特有的一些功能。

### 多态

函数本身就具备多态性，某一种事物有不同的具体的体现。

- 体现：父类引用或者接口的引用指向了自己的子类对象。//Animal a = new Cat();
- 多态的好处：提高了程序的扩展性。
- 多态的弊端：当父类引用指向子类对象时，虽然提高了扩展性，但是只能访问父类中具备的方法，不可以访问子类中特有的方法。(前期不能使用后期产生的功能，即访问的局限性)
- 多态的前提：
	- 1：必须要有关系，比如继承、或者实现。
	- 2：通常会有覆盖操作。

	
## 2. Java对象的生命周期

Java对象在虚拟机上运行有7阶段：创建、应用、不可见、不可达、收集、终结、对象空间重新分配。

<img src="../img/java_object_lifecycle.jpeg" style="zoom:100%"/>

1. 创建阶段(Created)

创建Java对象阶段的具体步骤如下：<br/>
①　为对象分配存储空间；<br/>
②　构造对象；<br/>
③　从超类到子类对static成员进行初始化，类的static成员的初始化在ClassLoader加载该类时进行；<br/>
④　超类成员变量按顺序初始化，递归调用超类的构造方法；<br/>
⑤　子类成员变量按顺序初始化，一旦对象被创建，子类构造方法就调用该对象并为某些变量赋值，完成后这个对象的状态就切换到了应用阶段。

2. 应用阶段(InUse)

对象至少被一个强引用持有。除非在系统中显式地使用了软引用、弱引用或虚引用。

3. 不可见阶段(Invisible)

处于不可见阶段的对象在虚拟机的对象引用根集合中再也找不到直接或间接的强引用，这些对象一般是所有线程栈中的临时变量。所有已经装载的静态变量或是对本地代码接口的引用。

当一个对象处于不可见阶段时，说明程序本身不再持有该对象的任何强引用，虽然该对象仍然存在。该对象可能被虚拟机中的某些已装载的静态变量线程或JNI等强引用持有，这些特殊的强引用称为“GC Root”。存在这些GC Root会导致对象的内存泄漏，无法被回收。

4. 不可达阶段(Unreachable)

对象处于不可达阶段是指该对象不再被任何强引用持有，回收器发现该对象已经不可达。

5. 收集阶段(Collected)

当垃圾回收器发现该对象已经处于“不可达阶段”并且垃圾回收器已经对该对象的内存空间重新分配做好准备时，对象进入“收集阶段”如果该对象已经重写了finalize()方法，则执行该方法的操作。

6. 终结阶段(Finalized)

当对象执行完finalize()方法后仍然处于不可达状态时，该对象进入终结阶段。在该阶段，等待垃圾回收器回收该对象空间。

7. 对象空间重新分配阶段(Deallocated)

若垃圾回收器对该对象占用的内存空间进行回收或者再分配，则该对象彻底消失，这个阶段称为“对象空间重新分配阶段”。

## 3. 抽象类和接口的区别

> 从设计层面来说，抽象是对类的抽象，是一种模板设计，接口是行为的抽象，是一种行为的规范。

### 抽象类与接口：
- 抽象类：一般用于描述一个体系单元，将一组共性内容进行抽取，特点：可以在类中定义抽象内容让子类实现，可以定义非抽象内容让子类直接使用。它里面定义的都是一些体系中的基本内容。
- 接口：一般用于定义对象的扩展功能，是在继承之外还需这个对象具备的一些功能。

抽象类和接口的共性：都是不断向上抽取的结果。

1. 接口的方法默认是public，所有方法在接口中不能有实现，抽象类可以有非抽象的方法
2. 接口中的实例变量默认是final类型的，而抽象类中则不一定
3. 一个类可以实现多个接口，但最多只能实现一个抽象类
4. 一个类实现接口的话要实现接口的所有方法，而抽象类不一定
5. 接口不能用new实例化，但可以声明，但是必须引用一个实现该接口的对象

## 常用关键字this、super、static和final

### this: 代表对象。就是所在函数所属对象的引用。

this到底代表什么呢？哪个对象调用了this所在的函数，this就代表哪个对象，就是哪个对象的引用。
开发时，什么时候使用this呢？
在定义功能时，如果该功能内部使用到了调用该功能的对象，这时就用this来表示这个对象。

this 还可以用于构造函数间的调用。
调用格式：this(实际参数)；
this对象后面跟上 .  调用的是成员属性和成员方法(一般方法)；
this对象后面跟上 () 调用的是本类中的对应参数的构造函数。

注意：用this调用构造函数，必须定义在构造函数的第一行。因为构造函数是用于初始化的，所以初始化动作一定要执行。否则编译失败。

### super关键字

super：代表是子类所属的父类中的内存空间引用。
	 注意：子父类中通常是不会出现同名成员变量的，因为父类中只要定义了，子类就不用在定义了，直接继承过来用就可以了。
	 
java中子类的所有构造函数中的第一行，其实都有一条隐身的语句super()，super(): 表示父类的构造函数，并会调用于参数相对应的父类中的构造函数。而super():是在调用父类中空参数的构造函数。


### static：关键字，是一个修饰符，用于修饰成员(成员变量和成员函数)。

特点：
  1. 想要实现对象中的共性数据的对象共享。可以将这个数据进行静态修饰。
  2. 被静态修饰的成员，可以直接被类名所调用。也就是说，静态的成员多了一种调用方式。类名.静态方式。
  3. 静态随着类的加载而加载。而且优先于对象存在。

弊端：
1. 有些数据是对象特有的数据，是不可以被静态修饰的。因为那样的话，特有数据会变成对象的共享数据。这样对事物的描述就出了问题。所以，在定义静态时，必须要明确，这个数据是否是被对象所共享的。
2. 静态方法只能访问静态成员，不可以访问非静态成员。
因为静态方法加载时，优先于对象存在，所以没有办法访问对象中的成员。
3.静态方法中不能使用this，super关键字。
因为this代表对象，而静态在时，有可能没有对象，所以this无法使用。

### 静态代码块：就是一个有静态关键字标示的一个代码块区域。定义在类中。
作用：可以完成类的初始化。静态代码块随着类的加载而执行，而且只执行一次（new 多个对象就只执行一次）。如果和主函数在同一类中，优先于主函数执行。

静态代码块、构造代码块、构造函数同时存在时的执行顺序：静态代码块 -> 构造代码块 -> 构造函数。

### final：关键字

在Java中，final关键字可以用来修饰类、方法和变量（包括成员变量和局部变量）。

- 修饰类

当用final修饰一个类时，表明这个类不能被继承。也就是说，如果一个类你永远不会让他被继承，就可以用final进行修饰。

- 修饰方法

下面这段话摘自《Java编程思想》第四版第143页：

　　“使用final方法的原因有两个。第一个原因是把方法锁定，以防任何继承类修改它的含义；第二个原因是效率。在早期的Java实现版本中，会将final方法转为内嵌调用。但是如果方法过于庞大，可能看不到内嵌调用带来的任何性能提升。在最近的Java版本中，不需要使用final方法进行这些优化了。“

　　因此，如果只有在想明确禁止 该方法在子类中被覆盖的情况下才将方法设置为final的。

注：类的private方法会隐式地被指定为final方法。

- 修饰变量

　　对于一个final变量，如果是基本数据类型的变量，则其数值一旦在初始化之后便不能更改；如果是引用类型的变量，则在对其初始化之后便不能再让其指向另一个对象。
　　
## 4. StringBuffer和StringBuilder的区别

1、StringBuffer 与 StringBuilder 中的方法和功能完全是等价的，

2、只是StringBuffer 中的方法大都采用了 synchronized 关键字进行修饰，因此是线程安全的，而 StringBuilder 没有这个修饰，可以被认为是线程不安全的。 

3、在单线程程序下，StringBuilder效率更快，因为它不需要加锁，不具备多线程安全，而StringBuffer则每次都需要判断锁，效率相对更低

- StringBuffer初始化及扩容机制

1.StringBuffer()的初始容量可以容纳16个字符，当该对象的实体存放的字符的长度大于16时，实体容量就自动增加。StringBuffer对象可以通过length()方法获取实体中存放的字符序列长度，通过capacity()方法来获取当前实体的实际容量。

2.StringBuffer(int size)可以指定分配给该对象的实体的初始容量参数为参数size指定的字符个数。当该对象的实体存放的字符序列的长度大于size个字符时，实体的容量就自动的增加。以便存放所增加的字符。

3.StringBuffer(String s)可以指定给对象的实体的初始容量为参数字符串s的长度额外再加16个字符。当该对象的实体存放的字符序列长度大于size个字符时，实体的容量自动的增加，以便存放所增加的字符。

## 5. Object类的equal和hashcode方法

因为hashCode()并不是完全可靠，有时候不同的对象他们生成的hashcode也会一样（生成hash值得公式可能存在的问题），所以hashCode()只能说是大部分时候可靠，并不是绝对可靠，所以我们可以得出：
- equal()相等的两个对象他们的hashCode()肯定相等，也就是用equal()对比是绝对可靠的。
- hashCode()相等的两个对象他们的equal()不一定相等，也就是hashCode()不是绝对可靠的。

boolean equals(Object obj)：用于比较两个对象是否相等，其实内部比较的就是两个对象地址。
而根据对象的属性不同，判断对象是否相同的具体内容也不一样。所以在定义类时，一般都会复写equals方法，建立本类特有的判断对象是否相同的依据。

int hashCode()：返回该对象的哈希码值。支持此方法是为了提高哈希表的性能。

###  hashCode（）与equals（）的相关规定
- 如果两个对象相等，则hashcode一定也是相同的
- 两个对象相等,对两个对象分别调用equals方法都返回true
- 两个对象有相同的hashcode值，它们也不一定是相等的
因此，equals 方法被覆盖过，则 hashCode 方法也必须被覆盖
- hashCode() 的默认行为是对堆上的对象产生独特值。如果没有重写 hashCode()，则该 class 的两个对象无论如何都不会相等（即使这两个对象指向相同的数据）

## 6. == 与 equals

**==** : 它的作用是判断两个对象的地址是不是相等。即，判断两个对象是不是同一个对象(“==”基本数据类型比较的是值，引用数据类型比较的是内存地址)。

**equals()** : 它的作用也是判断两个对象是否相等。但它一般有两种使用情况：
-  情况1：类没有覆盖 equals() 方法。则通过 equals() 比较该类的两个对象时，等价于通过“==”比较这两个对象。
- 情况2：类覆盖了 equals() 方法。一般，我们都覆盖 equals() 方法来两个对象的内容相等；若它们的内容相等，则返回 true (即，认为这两个对象相等)。
 
**举个例子：**

```java
public class Test {
    public static void main(String[] args) {
        String a = new String("ab"); // a 为一个引用
        String b = new String("ab"); // b为另一个引用,对象的内容一样
        String aa = "ab"; // 放在常量池中
        String bb = "ab"; // 从常量池中查找
        if (aa == bb) // true
            System.out.println("aa==bb");
        if (a == b) // false，非同一对象
            System.out.println("a==b");
        if (a.equals(b)) // true
            System.out.println("aEQb");
        if (42 == 42.0) { // true
            System.out.println("true");
        }
    }
}
```

**说明：**
- String 中的 equals 方法是被重写过的，因为 object 的 equals 方法是比较的对象的内存地址，而 String 的 equals 方法比较的是对象的值。
- 当创建 String 类型的对象时，虚拟机会在常量池中查找有没有已经存在的值和要创建的值相同的对象，如果有就把它赋给当前引用。如果没有就在常量池中重新创建一个 String 对象。

## 7. String为什么要设计成不可变的

1. 字符串常量池的需要

字符串常量池(String pool, String intern pool, String保留池) 是Java堆内存中一个特殊的存储区域, 当创建一个String对象时,假如此字符串值已经存在于常量池中,则不会创建一个新的对象,而是引用已经存在的对象。

2. 允许String对象缓存HashCode
Java中String对象的哈希码被频繁地使用, 比如在hashMap 等容器中。

字符串不变性保证了hash码的唯一性,因此可以放心地进行缓存.这也是一种性能优化手段,意味着不必每次都去计算新的哈希码. 在String类的定义中有如下代码:

``` java
private int hash;//用来缓存HashCode
```

3. 安全性
String被许多的Java类(库)用来当做参数,例如 网络连接地址URL,文件路径path,还有反射机制所需要的String参数等, 假若String不是固定不变的,将会引起各种安全隐患。

## 8.可否用try-catch捕获OOM以避免其发生

在try语句中声明了很大的对象，导致OOM，并且可以确认OOM是由try语句中的对象声明导致的，那么在catch语句中，可以释放掉这些对象，解决OOM的问题，继续执行剩余语句。但是这通常不是合适的做法。Java中管理内存除了显式地catch OOM之外还有更多有效的方法：比如SoftReference, WeakReference, 硬盘缓存等。在JVM用光内存之前，会多次触发GC，这些GC会降低程序运行的效率。如果OOM的原因不是try语句中的对象（比如内存泄漏），那么在catch语句中会继续抛出OOM

## 9. 单例设计模式
解决的问题：保证一个类在内存中的对象唯一性。
> 比如：多程序读取一个配置文件时，建议配置文件封装成对象。会方便操作其中数据，又要保证多个程序读到的是同一个配置文件对象，就需要该配置文件对象在内存中是唯一的。

如何保证对象唯一性呢？

思想：
- 1. 不让其他程序创建该类对象。
- 2. 在本类中创建一个本类对象。
- 3. 对外提供方法，让其他程序获取这个对象。


```java
    //饿汉式
class Single{
	private Single(){} //私有化构造函数。
        private static Single s = new Single(); //创建私有并静态的本类对象。
	    public static Single getInstance(){ //定义公有并静态的方法，返回该对象。
		    return s;
	    }
}

```

```java
    //懒汉式:延迟加载方式。
class Single2{
	private Single2(){}
        private static Single2 s = null;
            public static Single2 getInstance(){
                if(s==null)
                    s = new Single2();
                return s;
            }
}

```

## 10.内部类
如果A类需要直接访问B类中的成员，而B类又需要建立A类的对象。这时,为了方便设计和访问，直接将A类定义在B类中。就可以了。A类就称为内部类。内部类可以直接访问外部类中的成员。而外部类想要访问内部类，必须要建立内部类的对象。
```java
class Outer{
	int num = 4;	
	class  Inner {
		void show(){
			System.out.println("inner show run "+num);			
		}
	}
	public void method(){
		Inner in = new Inner();//创建内部类的对象。
		in.show();//调用内部类的方法。 
	}
}
```
当内部类定义在外部类中的成员位置上，可以使用一些成员修饰符修饰 private、static。
- 1：默认修饰符。
直接访问内部类格式：外部类名.内部类名 变量名 =  外部类对象.内部类对象;
Outer.Inner in = new Outer.new Inner();//这种形式很少用。
	但是这种应用不多见，因为内部类之所以定义在内部就是为了封装。想要获取内部类对象通常都通过外部类的方法来获取。这样可以对内部类对象进行控制。
- 2：私有修饰符。
	通常内部类被封装，都会被私有化，因为封装性不让其他程序直接访问。 
- 3：静态修饰符。
	如果内部类被静态修饰，相当于外部类，会出现访问局限性，只能访问外部类中的静态成员。
	注意；如果内部类中定义了静态成员，那么该内部类必须是静态的。

内部类编译后的文件名为：“外部类名$内部类名.java”；

为什么内部类可以直接访问外部类中的成员呢？
> 那是因为内部中都持有一个外部类的引用。这个是引用是 外部类名.this 
内部类可以定义在外部类中的成员位置上，也可以定义在外部类中的局部位置上。
当内部类被定义在局部位置上，只能访问局部中被final修饰的局部变量。

## 11. 多线程
进程：正在进行中的程序。其实进程就是一个应用程序运行时的内存分配空间。
线程：其实就是进程中一个程序执行控制单元，一条执行路径。
进程负责的是应用程序的空间的标示。线程负责的是应用程序的执行顺序。

一个进程至少有一个线程在运行，当一个进程中出现多个线程时，就称这个应用程序是多线程应用程序，每个线程在栈区中都有自己的执行空间，自己的方法区、自己的变量。
jvm在启动的时，首先有一个主线程，负责程序的执行，调用的是main函数。主线程执行的代码都在main方法中。
当产生垃圾时，收垃圾的动作，是不需要主线程来完成，因为这样，会出现主线程中的代码执行会停止，会去运行垃圾回收器代码，效率较低，所以由单独一个线程来负责垃圾回收。 

随机性的原理：因为cpu的快速切换造成，哪个线程获取到了cpu的执行权，哪个线程就执行。

返回当前线程的名称：Thread.currentThread().getName()
线程的名称是由：Thread-编号定义的。编号从0开始。
线程要运行的代码都统一存放在了run方法中。

线程要运行必须要通过类中指定的方法开启。start方法。（启动后，就多了一条执行路径）
start方法：1）、启动了线程；2）、让jvm调用了run方法。

## 12.为什么要有Runnable接口的出现
- 1：通过继承Thread类的方式，可以完成多线程的建立。但是这种方式有一个局限性，如果一个类已经有了自己的父类，就不可以继承Thread类，因为java单继承的局限性。
可是该类中的还有部分代码需要被多个线程同时执行。这时怎么办呢？
只有对该类进行额外的功能扩展，java就提供了一个接口Runnable。这个接口中定义了run方法，其实run方法的定义就是为了存储多线程要运行的代码。
所以，通常创建线程都用第二种方式。
因为实现Runnable接口可以避免单继承的局限性。

- 2：其实是将不同类中需要被多线程执行的代码进行抽取。将多线程要运行的代码的位置单独定义到接口中。为其他类进行功能扩展提供了前提。
所以Thread类在描述线程时，内部定义的run方法，也来自于Runnable接口。
 
实现Runnable接口可以避免单继承的局限性。而且，继承Thread，是可以对Thread类中的方法，进行子类复写的。但是不需要做这个复写动作的话，只为定义线程代码存放位置，实现Runnable接口更方便一些。所以Runnable接口将线程要执行的任务封装成了对象。

## 13.线程间通信：思路：多个线程在操作同一个资源，但是操作的动作却不一样。
1：将资源封装成对象。
2：将线程执行的任务(任务其实就是run方法。)也封装成对象。

### 等待唤醒机制：涉及的方法：
wait:将同步中的线程处于冻结状态。释放了执行权，释放了资格。同时将线程对象存储到线程池中。
notify：唤醒线程池中某一个等待线程。
notifyAll:唤醒的是线程池中的所有线程。

注意：
1：这些方法都需要定义在同步中。 
2：因为这些方法必须要标示所属的锁。
	你要知道 A锁上的线程被wait了,那这个线程就相当于处于A锁的线程池中，只能A锁的notify唤醒。
3：这三个方法都定义在Object类中。为什么操作线程的方法定义在Object类中？
	因为这三个方法都需要定义同步内，并标示所属的同步锁，既然被锁调用，而锁又可以是任意对象，那么能被任意对象调用的方法一定定义在Object类中。

### wait和sleep区别： 分析这两个方法：从执行权和锁上来分析：
- wait：可以指定时间也可以不指定时间。不指定时间，只能由对应的notify或者notifyAll来唤醒。
- sleep：必须指定时间，时间到自动从冻结状态转成运行状态(临时阻塞状态)。
- wait：线程会释放执行权，而且线程会释放锁。
- sleep：线程会释放执行权，但不是不释放锁。

线程的停止：通过stop方法就可以停止线程。但是这个方式过时了。
停止线程：原理就是：让线程运行的代码结束，也就是结束run方法。

怎么结束run方法？一般run方法里肯定定义循环。所以只要结束循环即可。
- 第一种方式：定义循环的结束标记。
- 第二种方式：如果线程处于了冻结状态，是不可能读到标记的，这时就需要通过Thread类中的interrupt方法，将其冻结状态强制清除。让线程恢复具备执行资格的状态，让线程可以读到标记，并结束。

## 14.Lock接口
> 多线程在JDK1.5版本升级时，推出一个接口Lock接口。
解决线程安全问题使用同步的形式，(同步代码块，要么同步函数)其实最终使用的都是锁机制。

到了后期版本，直接将锁封装成了对象。线程进入同步就是具备了锁，执行完，离开同步，就是释放了锁。
在后期对锁的分析过程中，发现，获取锁，或者释放锁的动作应该是锁这个事物更清楚。所以将这些动作定义在了锁当中，并把锁定义成对象。

所以同步是隐示的锁操作，而Lock对象是显示的锁操作，它的出现就替代了同步。

在之前的版本中使用Object类中wait、notify、notifyAll的方式来完成的。那是因为同步中的锁是任意对象，所以操作锁的等待唤醒的方法都定义在Object类中。

现在锁是指定对象Lock。所以查找等待唤醒机制方式需要通过Lock接口来完成。而Lock接口中并没有直接操作等待唤醒的方法，而是将这些方式又单独封装到了一个对象中。这个对象就是Condition，将Object中的三个方法进行单独的封装。并提供了功能一致的方法 await()、signal()、signalAll()体现新版本对象的好处。

```java
class BoundedBuffer {
   final Lock lock = new ReentrantLock();
   final Condition notFull  = lock.newCondition(); 
   final Condition notEmpty = lock.newCondition(); 
   final Object[] items = new Object[100];
   int putptr, takeptr, count;
   public void put(Object x) throws InterruptedException {
        lock.lock();
        try {
            while (count == items.length) 
                notFull.await();
            items[putptr] = x; 
            if (++putptr == items.length) putptr = 0;
            ++count;
            notEmpty.signal();
        } 
        finally {
            lock.unlock();
        }
   }
   public Object take() throws InterruptedException {
        lock.lock();
        try {
            while (count == 0) 
                notEmpty.await();
            Object x = items[takeptr]; 
            if (++takeptr == items.length) takeptr = 0;
            --count;
            notFull.signal();
            return x;
        } 
        finally {
            lock.unlock();
        }
   } 
 }

```

## 15. 对象的序列化

- 目的：将一个具体的对象进行持久化，写入到硬盘上。
- 注意：静态数据不能被序列化，因为静态数据不在堆内存中，是存储在静态方法区中。

如何将非静态的数据不进行序列化？
- 用transient 关键字修饰此变量即可。
- transient关键字只能修饰变量，而不能修饰方法和类。注意，本地变量是不能被transient关键字修饰的

Serializable：用于启动对象的序列化功能，可以强制让指定类具备序列化功能，该接口中没有成员，这是一个标记接口。这个标记接口用于给序列化类提供UID。这个uid是依据类中的成员的数字签名进行运行获取的。如果不需要自动获取一个uid，可以在类中，手动指定一个名称为serialVersionUID id号。依据编译器的不同，或者对信息的高度敏感性。最好每一个序列化的类都进行手动显示的UID的指定。

``` java
import java.io.*;
class ObjectStreamDemo {
	public static void main(String[] args) throws Exception{
		writeObj();
		readObj();
	}
	public static void readObj()throws Exception{
		ObjectInputStream ois = new ObjectInputStream(new FileInputStream("obj.txt"));
		Object obj = ois.readObject();//读取一个对象。
		System.out.println(obj.toString());
	}
	public static void writeObj()throws IOException{
		ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("obj.txt"));
		oos.writeObject(new Person("lisi",25)); //写入一个对象。
		oos.close();
	}
}
class Person implements Serializable{
	private static final long serialVersionUID = 42L;
	private transient String name;//用transient修饰后name将不会进行序列化
	public int age;
	Person(String name,int age){
		this.name = name;
		this.age = age;
	}
	public String toString(){
		return name+"::"+age;
	}
}
```

### transient使用细节——被transient关键字修饰的变量真的不能被序列化吗？
注意下面例子实现Externalizable接口，而不是Serilizable或Parcelable。

``` java
public class ExternalizableTest implements Externalizable {
 
    private transient String content = "是的，我将会被序列化，不管我是否被transient关键字修饰";
 
    @Override
    public void writeExternal(ObjectOutput out) throws IOException {
        out.writeObject(content);
    }
 
    @Override
    public void readExternal(ObjectInput in) throws IOException,
            ClassNotFoundException {
        content = (String) in.readObject();
    }
 
    public static void main(String[] args) throws Exception {
 
        ExternalizableTest et = new ExternalizableTest();
        ObjectOutput out = new ObjectOutputStream(new FileOutputStream(
                new File("test")));
        out.writeObject(et);
 
        ObjectInput in = new ObjectInputStream(new FileInputStream(new File(
                "test")));
        et = (ExternalizableTest) in.readObject();
        System.out.println(et.content);
 
        out.close();
        in.close();
    }
}

```
content变量会被序列化吗？好吧，我把答案都输出来了，是的，运行结果就是：

> 是的，我将会被序列化，不管我是否被transient关键字修饰

这是为什么呢，不是说类的变量被transient关键字修饰以后将不能序列化了吗？


我们知道在Java中，对象的序列化可以通过实现两种接口来实现，若实现的是Serializable接口，则所有的序列化将会自动进行，
- 若实现的是Externalizable接口，则没有任何东西可以自动序列化，需要在writeExternal方法中进行手工指定所要序列化的变量，这与是否被transient修饰无关。