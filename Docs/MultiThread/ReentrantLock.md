# ReentrantLock

可重入锁几点特性：

* 可重入
* 可中断 （可打破死锁）
* 可限时

### 可重入

由于ReentrantLock是重入锁，所以可以反复得到相同的一把锁，它有一个与锁相关的获取计数器，如果拥有锁的某个线程再次得到锁，那么获取计数器就加1，然后锁需要被释放两次才能获得真正释放(重入锁)。

```
lock.lock();
lock.lock();
try {
    i++;    
}           
finally {
    lock.unlock();
    lock.unlock();
}
```

### 可中断

对于上述死锁现象，可使用Reentrantlock的可中断特性解决。

思路是，使用Thread.interrupt()打断死锁的线程，释放锁资源，破坏死锁条件。

```
ReentrantLock.lockInterruptibly()

Acquires the lock unless the current thread is {Thread#interrupt} interrupted.
```

### 可限时

* 超时不能获得锁，就返回false，不会永久等待构成死锁
* 使用lock.tryLock(long timeout, TimeUnit unit)来实现可限时锁，参数为时间和单位。


```
public class TryLockTest extends Thread {

    public static ReentrantLock lock = new ReentrantLock();
    private String name;

    public TryLockTest(String name){
        super(name);
        this.name = name;
    }

    @Override
    public void run() {
        System.out.println(name + " run()");

        try {
            if (lock.tryLock(5, TimeUnit.SECONDS)) {
                Thread.sleep(10000);
            } else {
                System.out.println(this.getName() + " get lock failed");
            }
        } catch (Exception e) {
        } finally {
            if (lock.isHeldByCurrentThread()) {
                System.out.println("lock.isHeldByCurrentThread: " + this.getName());
                lock.unlock();
            }
        }
    }

    public static void main(String[] args) {
        TryLockTest t1 = new TryLockTest("TryLockTest 1");
        TryLockTest t2 = new TryLockTest("TryLockTest 2");

        t1.start();
        t2.start();
    }
}


输出：
TryLockTest 1 run()
TryLockTest 2 run()
TryLockTest 2 get lock failed	// 5秒后才输出，说明TryLockTest2等待5秒就不再等了，消除死锁
lock.isHeldByCurrentThread: TryLockTest 1

```