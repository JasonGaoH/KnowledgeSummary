# ThreadLocal

ThreadLocal为每一个使用该变量的线程都提供了独立的副本，可以做大线程间的数据隔离，每一个线程都可以访问各自内部的副本变量。

#### ThreadLocal的简单使用
```
import java.util.stream.IntStream;

public class ThreadLocalExample {
	public static void main(String[] args) {
        //创建ThreadLocal实例
		ThreadLocal<Integer> tlocal = new ThreadLocal<Integer>(); 
        //创建5个线程，使用tlocal
		IntStream.range(0, 5).forEach(i -> new Thread(() -> {
            //每个线程之间都是独立的，设置tlocal互相受影响
			tlocal.set(i);
			System.out.println(Thread.currentThread() + "set i " + tlocal.get());
			try {
				Thread.sleep(1000);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
			System.out.println(Thread.currentThread() + "get i " + tlocal.get());
		}).start()
		);
	}
}
```
上面的代码定义了一个全局唯一的ThreadLocal<Integer>,然后启动了5个线程对ThreadLocal进行set和get操作，通过下面的输出会发现5个线程之间彼此不会相互影响，每一个线程存入threadLocal中的值是完全不同彼此独立的。
```
Thread[Thread-0,5,main]set i 0
Thread[Thread-1,5,main]set i 1
Thread[Thread-3,5,main]set i 3
Thread[Thread-2,5,main]set i 2
Thread[Thread-4,5,main]set i 4
Thread[Thread-0,5,main]get i 0
Thread[Thread-1,5,main]get i 1
Thread[Thread-4,5,main]get i 4
Thread[Thread-2,5,main]get i 2
Thread[Thread-3,5,main]get i 3
```
 ThreadLocal除了set和get方法，还有一个initialValue的方法。

 initialValue()方法为ThreadLocal要保存的数据类型指定了一个初始化值，在ThreadLocal中默认返回值为null，实例代码如下：
 ```
 protected T initialValue() {
        return null;
    }
 ```
我么可以重写initialValue方法来进行数据的初始化，即可以不通过set方法来给ThreadLocal赋值，如下面的代码：
```
ThreadLocal<Object> threadLocal = new ThreadLocal<Object>() {
			@Override
			protected Object initialValue() {
				return new Object();
			}
		};
		
		new Thread(() -> {
			System.out.println(threadLocal.get());
		}).start();
		System.out.println(threadLocal.get());
```
数据输出如下，每一个想通过get方法获取到的值都是不一样的。
```
java.lang.Object@4f90c612
java.lang.Object@65ab7765
```
#### ThreadLocal的注意事项