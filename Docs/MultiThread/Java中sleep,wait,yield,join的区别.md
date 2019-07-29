- sleep()方法 
在指定时间内让当前正在执行的线程暂停执行，但不会释放“锁标志”。不推荐使用。 sleep()使当前线程进入阻塞状态，在指定时间内不会执行。

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

来看下Linux下os_linux.cpp中的sleep方法：
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
暂停当前正在执行的线程对象。
yield()只是使当前线程重新回到可执行状态，所以执行 yield()的线程有可能在进入到可执行 状态后马上又被执行。
yield()只能使同优先级或更高优先级的线程有执行的机会。?
调用 yield 方法并不会让线程进入阻塞状态，而是让线程重回就绪状态，它只需要等待重新 获取 CPU 执行时间，这一点是和 sleep 方法不一样的。
- join 方法
等待该线程终止。
等待调用 join 方法的线程结束，再继续执行。如:t.join();//主要用于等待 t 线程运行结束， 若无此句，main 则会执行完毕，导致结果不可预测。 在很多情况下，主线程创建并启动了线程，如果子线程中药进行大量耗时运算，主线程往往 将早于子线程结束之前结束。这时，如果主线程想等待子线程执行完成之后再结束，比如子 线程处理一个数据，主线程要取得这个数据中的值，就要用到 join()方法了。方法 join()的作 用是等待线程对象销毁。