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



死锁示例：

```
public class LockInterrupt extends Thread {

    public static ReentrantLock lock1 = new ReentrantLock();
    public static ReentrantLock lock2 = new ReentrantLock();

    private int lock;
    private String name;

    public LockInterrupt(int lock, String name) {
        super(name);
        this.name = name;
        this.lock = lock;
    }

    @Override
    public void run() {
        try {
            System.out.println(name + " run");

            if (lock == 1) {

            	//线程被interrupt时，会释放资源
                lock1.lockInterruptibly();
                System.out.println("lock1 block, lock1 locked");

                Thread.sleep(500);

                System.out.println("lock1 block, lock2 will lock");

                // 因为Thread1休眠了500ms，所以Thread2在这期间已经锁定了lock2，Thread1会在这句话等待，直到Thread2释放lock2
                lock2.lockInterruptibly();
                System.out.println("lock1 block, lock2 locked");

            } else {
                lock2.lockInterruptibly();
                System.out.println("lock2 block, lock2 locked");

                Thread.sleep(500);

                System.out.println("lock2 block, lock1 will lock");
                lock1.lockInterruptibly();
                System.out.println("lock2 block, lock1 locked");
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            System.out.println("finally");

            if (lock1.isHeldByCurrentThread()) {
                lock1.unlock();
            }
            if (lock2.isHeldByCurrentThread()) {
                lock2.unlock();
            }
            System.out.println(Thread.currentThread().getId() + ":线程退出");
        }
    }

    public static void main(String[] args) throws InterruptedException {
        LockInterrupt t1 = new LockInterrupt(1, "Thread 1");
        LockInterrupt t2 = new LockInterrupt(2, "Thread 2");
        t1.start();
        t2.start();
        Thread.sleep(1000);
    }   
}


输出：
Thread 1 run
Thread 2 run
lock1 block, lock1 locked
lock2 block, lock2 locked
lock1 block, lock2 will lock	// Thread1持有lock1, 等待lock2
lock2 block, lock1 will lock	// Thread2持有lock2, 等待lock1，造成死锁
```

使用jstack检查结果，也显示出现死锁：

```
Found one Java-level deadlock: 
 =============================
 
"Thread 2":
  waiting for ownable synchronizer 0x00000007958157e0, (a java.util.concurrent.locks.ReentrantLock$NonfairSync),
  which is held by "Thread 1"
"Thread 1":
  waiting for ownable synchronizer 0x0000000795815810, (a java.util.concurrent.locks.ReentrantLock$NonfairSync),
  which is held by "Thread 2"

Found 1 deadlock.

```

解决方案：

```
public static void main(String[] args) throws InterruptedException {
        ...

		// 中断线程，释放锁资源，打破死锁
        DeadlockChecker.check();
    }
    

  static class DeadlockChecker {

        private final static ThreadMXBean mbean = ManagementFactory.getThreadMXBean();

        public static void check() {
            Thread tt = new Thread(() -> {
                while (true) {
                    long[] deadlockedThreadIds = mbean.findDeadlockedThreads();
                    if (deadlockedThreadIds != null) {
                        ThreadInfo[] threadInfos = mbean.getThreadInfo(deadlockedThreadIds);
                        for (Thread t : Thread.getAllStackTraces().keySet()) {
                            for (int i = 0; i < threadInfos.length; i++) {
                                if (t.getId() == threadInfos[i].getThreadId()) {
                                    System.out.println(t.getName());
                                    
                                    // 打断死锁的关键，即中断线程
                                    t.interrupt();
                                }
                            }
                        }
                    }

                    try {
                        Thread.sleep(5000);
                    } catch (Exception e) {}
                }
            });
            tt.setDaemon(true);
            tt.start();
        }
    }
    
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