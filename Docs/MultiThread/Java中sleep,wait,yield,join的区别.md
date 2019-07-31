- sleep()方法 
sleep方法会使当前线程进入指定毫秒数的休眠，暂停执行，虽然给定了一个休眠的时间，但是最终要以系统的定时器和调度器的精准度为准，休眠有一个非常重要的特性，那就是不会放弃monitor锁的所有权。
sleep()使当前线程进入阻塞状态，在指定时间内不会执行。

#### Thread.java ->sleep，这是一个native方法。
```
  public static native void sleep(long millis) throws InterruptedException;
```
#### Thread.c ->JVM_Sleep
```
...
static JNINativeMethod methods[] = {
    {"start0",           "()V",        (void *)&JVM_StartThread},
    {"stop0",            "(" OBJ ")V", (void *)&JVM_StopThread},
    {"isAlive",          "()Z",        (void *)&JVM_IsThreadAlive},
    {"suspend0",         "()V",        (void *)&JVM_SuspendThread},
    {"resume0",          "()V",        (void *)&JVM_ResumeThread},
    {"setPriority0",     "(I)V",       (void *)&JVM_SetThreadPriority},
    {"yield",            "()V",        (void *)&JVM_Yield},
    {"sleep",            "(J)V",       (void *)&JVM_Sleep},
    {"currentThread",    "()" THD,     (void *)&JVM_CurrentThread},
    {"countStackFrames", "()I",        (void *)&JVM_CountStackFrames},
    {"interrupt0",       "()V",        (void *)&JVM_Interrupt},
    {"isInterrupted",    "(Z)Z",       (void *)&JVM_IsInterrupted},
    {"holdsLock",        "(" OBJ ")Z", (void *)&JVM_HoldsLock},
    {"getThreads",        "()[" THD,   (void *)&JVM_GetAllThreads},
    {"dumpThreads",      "([" THD ")[[" STE, (void *)&JVM_DumpThreads},
    {"setNativeName",    "(" STR ")V", (void *)&JVM_SetNativeThreadName},
};
...

```

#### jvm.cpp -> JVM_Sleep
```
JVM_ENTRY(void, JVM_Sleep(JNIEnv* env, jclass threadClass, jlong millis))
  JVMWrapper("JVM_Sleep");

  if (millis < 0) {
    THROW_MSG(vmSymbols::java_lang_IllegalArgumentException(), "timeout value is negative");
  }

  if (Thread::is_interrupted (THREAD, true) && !HAS_PENDING_EXCEPTION) {
    THROW_MSG(vmSymbols::java_lang_InterruptedException(), "sleep interrupted");
  }

  // Save current thread state and restore it at the end of this block.
  // And set new thread state to SLEEPING.
  JavaThreadSleepState jtss(thread);

  HS_DTRACE_PROBE1(hotspot, thread__sleep__begin, millis);

  if (millis == 0) {
    // When ConvertSleepToYield is on, this matches the classic VM implementation of
    // JVM_Sleep. Critical for similar threading behaviour (Win32)
    // It appears that in certain GUI contexts, it may be beneficial to do a short sleep
    // for SOLARIS
    if (ConvertSleepToYield) {
      os::yield();
    } else {
      ThreadState old_state = thread->osthread()->get_state();
      thread->osthread()->set_state(SLEEPING);
      os::sleep(thread, MinSleepInterval, false);
      thread->osthread()->set_state(old_state);
    }
  } else {
    ThreadState old_state = thread->osthread()->get_state();
    thread->osthread()->set_state(SLEEPING);
    if (os::sleep(thread, millis, true) == OS_INTRPT) {
      // An asynchronous exception (e.g., ThreadDeathException) could have been thrown on
      // us while we were sleeping. We do not overwrite those.
      if (!HAS_PENDING_EXCEPTION) {
        HS_DTRACE_PROBE1(hotspot, thread__sleep__end,1);
        // TODO-FIXME: THROW_MSG returns which means we will not call set_state()
        // to properly restore the thread state.  That's likely wrong.
        THROW_MSG(vmSymbols::java_lang_InterruptedException(), "sleep interrupted");
      }
    }
    thread->osthread()->set_state(old_state);
  }
  HS_DTRACE_PROBE1(hotspot, thread__sleep__end,0);
JVM_END 

```
在JVM_Sleep方法中，通过thread->osthread()->get_state()获取 OSThread 对象，并将其状态设置为SLEEPING等到 sleep 结束后设置回原来的状态。

#### sleep方法是在os.hpp头文件中定义：
```
static int sleep(Thread* thread, jlong ms, bool interruptable);
```
sleep这个操作跟具体的操作系统实现有关，
linux对应目录是：jdk-8-hotspot/src/os/linux/vm/os_linux.cpp；
windows对应的目录是：jdk-8-hotspot/src/os/windows/vm/os_windows.cpp；

来看下Linux下os_windowscpp中的sleep方法：
```
int os::sleep(Thread* thread, jlong ms, bool interruptable) {
  jlong limit = (jlong) MAXDWORD;

  while(ms > limit) {
    int res;
    if ((res = sleep(thread, limit, interruptable)) != OS_TIMEOUT)
      return res;
    ms -= limit;
  }

  assert(thread == Thread::current(),  "thread consistency check");
  OSThread* osthread = thread->osthread();
  OSThreadWaitState osts(osthread, false /* not Object.wait() */);
  int result;
  if (interruptable) {
    assert(thread->is_Java_thread(), "must be java thread");
    JavaThread *jt = (JavaThread *) thread;
    ThreadBlockInVM tbivm(jt);

    jt->set_suspend_equivalent();
    // cleared by handle_special_suspend_equivalent_condition() or
    // java_suspend_self() via check_and_wait_while_suspended()

    HANDLE events[1];
    events[0] = osthread->interrupt_event();
    HighResolutionInterval *phri=NULL;
    if(!ForceTimeHighResolution)
      phri = new HighResolutionInterval( ms );
    if (WaitForMultipleObjects(1, events, FALSE, (DWORD)ms) == WAIT_TIMEOUT) {
      result = OS_TIMEOUT;
    } else {
      ResetEvent(osthread->interrupt_event());
      osthread->set_interrupted(false);
      result = OS_INTRPT;
    }
    delete phri; //if it is NULL, harmless

    // were we externally suspended while we were waiting?
    jt->check_and_wait_while_suspended();
  } else {
    assert(!thread->is_Java_thread(), "must not be java thread");
    Sleep((long) ms);
    result = OS_TIMEOUT;
  }
  return result;
}
```
获取 OSThread 对象，然后通过 OSThreadWaitState 设置线程状态为等待，修改操作分别在构造函数和析构函数中实现。

#### sleep方法中的ThreadBlockInVM
前面说到 ThreadBlockInVM 会检查当前线程用不用进入 safepoint，它主要的逻辑如下：

首先设置 Java 线程状态，将状态加一，由_thread_in_vm = 6变为_thread_in_vm_trans = 7，从“运行vm本身代码”到“相应的过度状态”。
os::is_MP()用于判断计算机系统是否为多核系统，多核情况下需要做内存屏障处理，这是为了让每个线程都能实时同步状态。
内存屏障有两种方式，一种是rderAccess::fence()，它的实现是直接通过CPU指令来实现，汇编指令为__asm__ volatile ("lock; addl $0,0(%%rsp)" : : : "cc", "memory");，这种方式代价比较大。而另外一种为InterfaceSupport::serialize_memory，由 JVM 模拟实现，效率高一点。
调用SafepointSynchronize::block尝试在该安全点进行阻塞。
设置 Java 线程状态为_thread_blocked，即阻塞。


- wait()方法
在其他线程调用对象的 notify 或 notifyAll 方法前，导致当前线程等待。线程会释放掉它所占有的“锁标志”，从而使别的线程有机会抢占该锁。当前线程必须拥有当前对象锁。如果当前 线程不是此锁的拥有者，会抛出 IllegalMonitorStateException 异常。唤醒当前对象锁的等待 线程使用 notify 或 notifyAll 方法，也必须拥有相同的对象锁，否则也会抛出 IllegalMonitorStateException 异常。
wait()和 notify()必须在 synchronized 函数或 synchronized block 中进行调用。如果在 non- synchronized 函数或 non-synchronized block 中进行调用，虽然能编译通过，但在运行时 会发生 IllegalMonitorStateException 的异常。

#### wait方法的调用流程

wait是Java中Object.java中的方法,这个后面调用的是wait这个native方法。
```
public final void wait() throws InterruptedException {
    wait(0);
}

public final native void wait(long timeout) throws InterruptedException;
```

Java层声明的这个wait方法在Object.c中，wait同样也是一个注册到JVM中的方法，具体的实现在JVM_MonitorWait中。

```
...
static JNINativeMethod methods[] = {
    {"hashCode",    "()I",                    (void *)&JVM_IHashCode},
    {"wait",        "(J)V",                   (void *)&JVM_MonitorWait},
    {"notify",      "()V",                    (void *)&JVM_MonitorNotify},
    {"notifyAll",   "()V",                    (void *)&JVM_MonitorNotifyAll},
    {"clone",       "()Ljava/lang/Object;",   (void *)&JVM_Clone},
};
...

```
jvm.cpp中的JVM_MonitorWait  
```
JVM_ENTRY(void, JVM_MonitorWait(JNIEnv* env, jobject handle, jlong ms))
  JVMWrapper("JVM_MonitorWait");
  Handle obj(THREAD, JNIHandles::resolve_non_null(handle));
  assert(obj->is_instance() || obj->is_array(), "JVM_MonitorWait must apply to an object");
  JavaThreadInObjectWaitState jtiows(thread, ms != 0);
  if (JvmtiExport::should_post_monitor_wait()) {
    JvmtiExport::post_monitor_wait((JavaThread *)THREAD, (oop)obj(), ms);
  }
  ObjectSynchronizer::wait(obj, ms, CHECK);
JVM_END
```

接着调用objectMonitor.cpp中的void ObjectMonitor::wait(jlong millis, bool interruptible, TRAPS)方法。

这个wait方法太长，就不贴代码了。

- yield 方法
yield属于一种启发式方法，岂会提醒调度器我愿意放弃当前CPU资源，如果CPU资源不紧张，则会忽略这种提醒。

```
import java.util.function.IntFunction;
import java.util.stream.IntStream;

public class ThreadYield {
	public static void main(String[] args) {
		IntStream.range(0, 2).mapToObj(new IntFunction<Thread>(){
			@Override
			public Thread apply(int value) {
				return ThreadYield.create(value);
			}}).forEach(new Consumer<Thread>() {
				@Override
				public void accept(Thread t) {
					t.start();
				}
			});
	}
	
	private static Thread create(int index) {
		return new Thread(){
			@Override
			public void run() {
				//if(index == 0) Thread.yield();
				System.out.println(index);
			}
		};
	}
}
```

上面的程序运行很多次,你会发现输出的结果不一致，有时候是0最先打印出来，有时候是1最先打印出来，但是当你打开代码的注释部分，你会发现，顺序始终是0，1。

因为第一个线程如果先获得了CPU资源，它会比较谦虚，主动告诉CPU调度器释放了原本属于自己的资源，但是yield知识一个提示，CPU调度器并不会担保每次都能满足yield提示。

调用 yield 方法并不会让线程进入阻塞状态，而是让线程重回就绪状态，它只需要等待重新 获取 CPU 执行时间，这一点是和 sleep 方法不一样的。

- join 方法
join某个线程A，会使当前线程B进入等待，直到线程A结束生命周期，或者到达给定的时间，那么在此期间B线程是处于BLOCKED的，而不是A线程。

```
import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.TimeUnit;
import java.util.function.Consumer;
import java.util.function.IntFunction;
import java.util.stream.IntStream;

public class ThreadJoin {

	public static void main(String[] args) throws InterruptedException {
		List<Thread> threads = new ArrayList<Thread>();
		IntStream.range(0, 2).mapToObj(new IntFunction<Thread>(){
			@Override
			public Thread apply(int value) {
				return ThreadJoin.create(value);
			}}).forEach(new Consumer<Thread>() {
				@Override
				public void accept(Thread t) {
					t.start();
					threads.add(t);
				}
			});
		
		for(Thread thread:threads) {
			thread.join();
		}
		for(int i =0;i<3;i++) {
			System.out.println(Thread.currentThread().getName() + "#" + i);
			shortSleep();
		}
	}
	
	private static Thread create(int seq) {
		return new Thread(seq + ""){
			@Override
			public void run() {
				for(int i=0;i< 3;i++) {
					System.out.println(Thread.currentThread().getName() + "#" + i);
					shortSleep();
				}
			}
		};
	}
	
	private static void shortSleep() {
		try {
			TimeUnit.SECONDS.sleep(1);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
	}
}
```

运行上面的程序你就会发现线程1和线程2交替地输出直到它们生命周期结束，main线程的循环才会开始执行，程序输出如下：
```
0#0
1#0
0#1
1#1
0#2
1#2
main#0
main#1
main#2
```
如果将join部分的代码注释掉，那么三个线程会交替地输出，输出如下：
```
0#0
main#0
1#0
1#1
0#1
main#1
0#2
1#2
main#2
```

#### interrupt方法

如下方法会使得当前线程进入阻塞状态，而调用线程的interrupt方法会打断阻塞。
- Object的wait方法
- Object的wait(long)方法
- Thread的sleep(long)方法
- Thread的join方法
- InterruptibleChannel的io操作
- Selector的wakeup方法
- 其他方法

上述若干方法都会使得当前线程进入阻塞状态，若另外的一个线程调用被阻塞线程的interrupt方法，则会打断这种阻塞，因此这种方法有时会被称为可中断方法，但是，打断一个线程并不等于该线程生命周期会结束，仅仅是打断了当前线程的阻塞状态。

一旦线程在阻塞的情况下被打断，都会抛出一个称为InterruptException的异常，这个异常就像一个signal(信号)一样通知当前线程被打断了。

interrupt这个方法到了做了什么样的事情呢？在一个线程内存存在着名为interrupt flag的标识，如果一个线程被interrupt，那么他的flag将被设置为true，但是如果当前线程正在执行可中断方法被阻塞时，调用interrupt方法将其中断，这个时候它的flag会被清除，清除即设置为false。

具体可以看下面的程序代码：
```
import java.util.concurrent.TimeUnit;

public class ThreadInterrupt {
	public static void main(String[] args) throws InterruptedException {
		Thread thread = new Thread(){
			@Override
			public void run() {
				while(true) {
					//loop
				}
			}
		};
		thread.start();
		TimeUnit.MILLISECONDS.sleep(2);
		System.out.println("Thread is interrupted:  " + thread.isInterrupted());
		thread.interrupt();
		System.out.println("Thread is interrupted:  " + thread.isInterrupted());
	}
}

```
输出结果如下：
```
Thread is interrupted:  false
Thread is interrupted:  true

```

上面的run方法中我们写了个死循环，这里没有使用sleep方法，因为sleep是可中断方法，可中断方法在捕获InterruptException后会将线程interrupt复位。

我们把程序改成下面这样。
```
import java.util.concurrent.TimeUnit;

public class ThreadInterrupt {

	public static void main(String[] args) throws InterruptedException {
		Thread thread = new Thread(){
			@Override
			public void run() {
				while(true) {
					try {
						TimeUnit.MINUTES.sleep(2);
					} catch (InterruptedException e) {
						System.out.println("I am be interrupted:  " + isInterrupted());
					}
				}
			}
		};
		thread.setDaemon(true);
		thread.start();
		TimeUnit.MILLISECONDS.sleep(2);
		System.out.println("Thread is interrupted:  " + thread.isInterrupted());
		thread.interrupt();
		System.out.println("Thread is interrupted:  " + thread.isInterrupted());
	}

}

```
由于在run方法中使用了sleep这个可中断方法，它会捕获到中断信号，并且会擦除interrupt表示，因此程序的执行结果都会是false。
```
Thread is interrupted:  false
I am be interrupted:  false
Thread is interrupted:  false

```

#### interrupted

interrupted是一个静态方法，虽然其页用于判断当前线程是否被中断，但是它和成员方法isInterrupted还是有很大区别的，调用该方法会直接擦除的线程的interrupt标识，需要注意的是，如果当前线程被打断了那么第一次调用interrupted方法会返回true，并且立即擦除了interrupt标识，第二次包括以后的调用永远都会放回false，除非在此期间有一次地被打断。

```
import java.util.concurrent.TimeUnit;

public class ThreadInterrupt {

	public static void main(String[] args) throws InterruptedException {
		Thread thread = new Thread(){
			@Override
			public void run() {
				while(true) {
					System.out.println(Thread.interrupted());
				}
			}
		};
		thread.setDaemon(true);
		thread.start();
		TimeUnit.MILLISECONDS.sleep(2);
		thread.interrupt();			
	}

```

由于不想要受到可中断方法如sleep的影响，在Thread的run方法中没有进行任何短暂的休眠。
在很多false中出现了一个true，也就是interrupted方法判断到了其被中断，立即擦除了中断标识。

```
...
false
false
true
false
false
...

```