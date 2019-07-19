### 注解相当于是加在代码上的标签。

####  注解的定义

```
public @interface Test {
}
```
创建形式跟接口类似，在interface前需要加"@"符号

#### 元注解
为什么需要元注解？元注解是用来干嘛的？

首先元注解是可以注解到注解上的注解，或者说元注解是一种基本注解，但是它能够应用到其它的注解上面。

因为注解定义出来，这个注解的作用域是什么，也就是说这个注解将应用于什么地方（例如是一个方法或者一个字段上）。另外，注解定义出来，需要明确注解是哪一个级别可用，在源代码中（SOURCE）、类文件中（CLASS）或者运行时（RUNTIME）。

这也就是为什么需要元注解，元注解就是帮助定义一下注解的是怎么使用的。

元注解有@Retention、@Documented、@Target、@Inherited、@Repeatable 5 种。

#### @Retention
Retention 的英文意为保留期的意思。当 @Retention 应用到一个注解上的时候，它解释说明了这个注解的的存活时间。

它的取值如下：

RetentionPolicy.SOURCE 注解只在源码阶段保留，在编译器进行编译时它将被丢弃忽视。
RetentionPolicy.CLASS 注解只被保留到编译进行的时候，它并不会被加载到 JVM 中。
RetentionPolicy.RUNTIME 注解可以保留到程序运行的时候，它会被加载进入到 JVM 中，所以在程序运行时可以获取到它们。

```
@Retention(RetentionPolicy.RUNTIME)
public @interface Test {
}
```
上面的代码中，我们指定Test注解可以在程序运行周期被获取到，因此它的生命周期非常的长。

这里对于注解的@Retention怎么使用还是有点疑问，什么时候该用Source，什么时候该用CLASS以及什么时候该用RUNTIME呢？

根据主键的存活时间来划分如下
首先要明确存活的时间长度 SOURCE < CLASS < RUNTIME ，所以前者能作用的地方后者一定也能作用。

举例，一般如果需要在运行时去动态获取注解信息，那只能用 RUNTIME 注解，比如Android中使用@Deprecated标记方法，字段，类等是否废弃，
还有很多场景也会使用Runtime，例如dagger2中的Component注解，dagger2中会在运行时会通过这个注解有些特殊的行为交互。

如果要在编译时进行一些预处理操作，比如生成一些辅助代码（如 ButterKnife），就用 CLASS注解。

例如AndroidX包下面的DrawableRes。
```
@Documented
@Retention(RetentionPolicy.CLASS)
@Target({ElementType.METHOD, ElementType.PARAMETER, ElementType.FIELD, ElementType.LOCAL_VARIABLE})
public @interface DrawableRes {
}
```
DrawableRes表示期望整数参数、字段或方法返回值是一个drawable资源引用。

如果只是做一些检查性的操作，比如 @Override 和 @SuppressWarnings，使用SOURCE 注解。

#### @Documented
顾名思义，这个元注解肯定是和文档有关。它的作用是能够将注解中的元素包含到 Javadoc 中去。

#### @Target
Target 是目标的意思，@Target 指定了注解运用的地方。

你可以这样理解，当一个注解被 @Target 注解时，这个注解就被限定了运用的场景。

@Target 有下面的取值

- ElementType.ANNOTATION_TYPE 可以给一个注解进行注解
- ElementType.CONSTRUCTOR 可以给构造方法进行注解
- ElementType.FIELD 可以给属性进行注解
- ElementType.LOCAL_VARIABLE 可以给局部变量进行注解
- ElementType.METHOD 可以给方法进行注解
- ElementType.PACKAGE 可以给一个包进行注解
- ElementType.PARAMETER 可以给一个方法内的参数进行注解
- ElementType.TYPE 可以给一个类型进行注解，比如类、接口、枚举


#### @Inherited
Inherited 是继承的意思，但是它并不是说注解本身可以继承，而是说如果一个超类被 @Inherited 注解过的注解进行注解的话，那么如果它的子类没有被任何注解应用的话，那么这个子类就继承了超类的注解。

```
@Inherited
@Retention(RetentionPolicy.RUNTIME)
@interface Test {}

@Test
public class A {}

public class B extends A {}

```
注解 Test 被 @Inherited 修饰，之后类 A 被 Test 注解，类 B 继承 A,类 B 也拥有 Test 这个注解。

@Repeatable
Repeatable 自然是可重复的意思。@Repeatable 是 Java 1.8 才加进来的，所以算是一个新的特性。

什么样的注解会多次应用呢？通常是注解的值可以同时取多个。

举个例子，一个人他既是程序员又是产品经理,同时他还是个画家。

```
@interface Persons {
	Person[]  value();
}

@Repeatable(Persons.class)
@interface Person{
	String role default "";
}

@Person(role="artist")
@Person(role="coder")
@Person(role="PM")
public class SuperMan{
	
}
```

#### 注解的属性
注解的属性也叫做成员变量。注解只有成员变量，没有方法。注解的成员变量在注解的定义中以“无形参的方法”形式来声明，其方法名定义了该成员变量的名字，其返回值定义了该成员变量的类型。

```
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface Test {
	int id();
	String msg();
}

```
上面代码定义了 Test 这个注解中拥有 id 和 msg 两个属性。在使用的时候，我们应该给它们进行赋值。

赋值的方式是在注解的括号内以 value="" 形式，多个属性之前用 ，隔开。

```
@Test(id=3,msg="hello annotation")
public class Test {

}

```

需要注意的是，在注解中定义属性时它的类型必须是 8 种基本数据类型外加 类、接口、注解及它们的数组。

注解中属性可以有默认值，默认值需要用 default 关键值指定。比如：

```
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface TestAnnotation {
	public int id() default -1;
	public String msg() default "Hi";
}

```

#### 注解的提取

要想正确检阅注解，离不开一个手段，那就是反射。

#### 注解与反射。
注解通过反射获取。首先可以通过 Class 对象的 isAnnotationPresent() 方法判断它是否应用了某个注解。
```
public boolean isAnnotationPresent(Class<? extends Annotation> annotationClass) {}

```

然后通过 getAnnotation() 方法来获取 Annotation 对象。
```
 public <A extends Annotation> A getAnnotation(Class<A> annotationClass) {}

```
或者是 getAnnotations() 方法。
```
public Annotation[] getAnnotations() {}
```

```
@TestAnnotation()
public class Test {
	public static void main(String[] args) {
		boolean hasAnnotation = Test.class.isAnnotationPresent(TestAnnotation.class);
		if ( hasAnnotation ) {
			TestAnnotation testAnnotation = Test.class.getAnnotation(TestAnnotation.class);
			System.out.println("id:"+testAnnotation.id());
			System.out.println("msg:"+testAnnotation.msg());
		}
	}
}
```

程序的运行结果是：
```
id:-1
msg:
```

#### apt技术

[link](https://blog.csdn.net/briblue/article/details/73824058)