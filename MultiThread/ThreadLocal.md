# ThreadLocal

ThreadLocal是一个可以创建线程局部变量的类。

通常情况下，多个线程虽然共享同一个对象，其中一个线程修改了该共享对象的内容，其他线程访问时将获得修改后的值。但是，如果用ThreadLocal修饰该共享的对象，每个线程对该变量的修改，不会影响到其他线程。即其他线程访问时，还是修改前的值。就好像ThreadLocal为每个线程创建了一份私有副本一样。因此，可以用ThreadLocal实现线程安全。


Example1：

没有使用ThreadLocal时，线程间对同一个对象的修改，将会互相影响。

```
public class ThreadLocalExample {

    public static void main(String[] args) {
		
        CommonRunnable commonRunnable = new CommonRunnable();

        Thread threadA = new Thread(commonRunnable);
        Thread threadB = new Thread(commonRunnable);

        threadA.start();
        threadB.start();

        try {
            threadA.join(); //wait for threadA to terminate
            threadB.join(); //wait for threadB to terminate
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    public static class CommonRunnable implements Runnable {

		// 因为CommonRunnable是静态类，所以一个线程修改value，其他线程获取修改后的值
        private int value;

        @Override
        public void run() {
            value =  (int) (Math.random() * 100D);
            System.out.println(Thread.currentThread().getName() + ": set value " + value);

            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
            }

            System.out.println(Thread.currentThread().getName() + ": " + value);
        }
    }
}


输出：
Thread-0: set value 62
Thread-1: set value 86	//Thread-1将value修改为86
Thread-0: 86
Thread-1: 86			//Thread-0和Thread-1获得的都是86

```

Example2：

使用ThreadLocal时，线程对同一个对象的修改，不会影响其他线程。

```
public class ThreadLocalExample {

    public static void main(String[] args) {
        ThreadLocalRunnable threadLocalRunnable = new ThreadLocalRunnable();

        Thread threadA = new Thread(threadLocalRunnable);
        Thread threadB = new Thread(threadLocalRunnable);

        threadA.start();
        threadB.start();

        try {
            threadA.join(); //wait for threadA to terminate
            threadB.join(); //wait for threadB to terminate
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    public static class ThreadLocalRunnable implements Runnable {

        private ThreadLocal<Integer> threadLocal = new ThreadLocal<Integer>();

        @Override
        public void run() {
            int value =  (int) (Math.random() * 100D);
            threadLocal.set(value);
            System.out.println(Thread.currentThread().getName() + ": set value " + value);

            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
            }

            System.out.println(Thread.currentThread().getName() + ": " + threadLocal.get());
        }
    }
    



输出：
Thread-1: set value 93
Thread-0: set value 55
Thread-1: 93	//虽然Thread-0先把value修改为55，但是对Thread-1没有影响
Thread-0: 55

```


### ThreadLocal实现原理

大家应该很好奇，上例Example2中是如何做到一个线程对共享对象的修改，不会影响到其他线程。

看下ThreadLocal的set()的实现：

* 首先获取当前线程
* 获取当前线程的成员threadLocals，该成员是ThreadLocalMap的实例
* 如果上述ThreadLocalMap对象不为空，则设置值，否则创建这个ThreadLocalMap对象并设置值

源码如下:

```
public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
}

ThreadLocalMap getMap(Thread t) {
    return t.threadLocals;
}

void createMap(Thread t, T firstValue) {
    t.threadLocals = new ThreadLocalMap(this, firstValue);
}
```


看下Thread.threadLocals变量

```
class Thread implements Runnable {
    /* ThreadLocal values pertaining to this thread. This map is maintained
     * by the ThreadLocal class. */

    ThreadLocal.ThreadLocalMap threadLocals = null;
}
```

总结：放入ThreadLocal的值，其实是放入了当前线程的一个私有成员ThreadLocalMap中，因为ThreadLocalMap只属于当前线程，所以，相当于ThreadLocal为每个线程制造了一份共享变量的副本。从而达到线程安全。



##### 参考文章
* [Java ThreadLocal](http://tutorials.jenkov.com/java-concurrency/threadlocal.html)
* [理解Java中的ThreadLocal](https://droidyue.com/blog/2016/03/13/learning-threadlocal-in-java/)
