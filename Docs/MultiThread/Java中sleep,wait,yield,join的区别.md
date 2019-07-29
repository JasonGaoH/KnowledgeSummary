- sleep()方法 
在指定时间内让当前正在执行的线程暂停执行，但不会释放“锁标志”。不推荐使用。 sleep()使当前线程进入阻塞状态，在指定时间内不会执行。
- wait()方法
在其他线程调用对象的 notify 或 notifyAll 方法前，导致当前线程等待。线程会释放掉它所占有的“锁标志”，从而使别的线程有机会抢占该锁。当前线程必须拥有当前对象锁。如果当前 线程不是此锁的拥有者，会抛出 IllegalMonitorStateException 异常。唤醒当前对象锁的等待 线程使用 notify 或 notifyAll 方法，也必须拥有相同的对象锁，否则也会抛出 IllegalMonitorStateException 异常。
wait()和 notify()必须在 synchronized 函数或 synchronized block 中进行调用。如果在 non- synchronized 函数或 non-synchronized block 中进行调用，虽然能编译通过，但在运行时 会发生 IllegalMonitorStateException 的异常。
- yield 方法
暂停当前正在执行的线程对象。
yield()只是使当前线程重新回到可执行状态，所以执行 yield()的线程有可能在进入到可执行 状态后马上又被执行。
yield()只能使同优先级或更高优先级的线程有执行的机会。?
调用 yield 方法并不会让线程进入阻塞状态，而是让线程重回就绪状态，它只需要等待重新 获取 CPU 执行时间，这一点是和 sleep 方法不一样的。
- join 方法
等待该线程终止。
等待调用 join 方法的线程结束，再继续执行。如:t.join();//主要用于等待 t 线程运行结束， 若无此句，main 则会执行完毕，导致结果不可预测。 在很多情况下，主线程创建并启动了线程，如果子线程中药进行大量耗时运算，主线程往往 将早于子线程结束之前结束。这时，如果主线程想等待子线程执行完成之后再结束，比如子 线程处理一个数据，主线程要取得这个数据中的值，就要用到 join()方法了。方法 join()的作 用是等待线程对象销毁。