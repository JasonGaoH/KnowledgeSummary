1. 什么是AsyncTask

*AsyncTask 即 asynchronous task，异步任务*。

AsyncTask实际上是围绕Thread和Handler设计的一个辅助类，在内部是对Thread和Handler的一种封装。AsyncTask的异步体现在由后台线程进行运算（访问网络等比较耗时的操作），然后将结果发布到用户界面上来更新UI，使用AsyncTask使得我不用操作Thread和Handler。

2. AsyncTask的简单使用

```
new AsyncTask<String,String,String>(){
    //// 运行在主线程中，做预备工作/////
    onPreExecute(){
    
    }
    // 运行在子线程中，做耗时操作
    String doingBackGround(String s){
    
    }
    // 运行在主线程中，耗时操作完成，更新UI
    onPostExecute(String s){
    			
    }

}.execute(String);
```
AsyncTask用法比较简单，Google设计这个类就是为了方便我们进行类似Handler这样的异步操作。

##### 如上代码，一般使用AsyncTask只要重写里面的三个方法，onPreExecute和onPostExecute不是抽象方法，不是必须实现，实现这两种方法一般能让代码的逻辑更加清晰。onPreExecute运行在主线程中，做一些准备工作。onPostExecute同样运行在主线程中，用于在耗时操作完成后，更新UI。另外，还有一个onProgressUpdate方法，用于在后台任务执行过程中来实时地更新UI。


##### doingBackGround则是抽象方法，必须实现，我们使用AsyncTask时希望将一些耗时操作放在子线程中，doingBackGround中逻辑就相当于我们在Thread-Handler中Thread中的run方法中实现的逻辑。

##### execute方法用于启动执行任务

3. 从源码角度看AsyncTask的设计
接下来进入这篇文章的重点，我们从源码角度来分析AsyncTask是如何实现的。
>#### AsyncTask的execute方法

```
public final AsyncTask<Params, Progress, Result> execute(Params... params) {
        return executeOnExecutor(sDefaultExecutor, params);
    }
```
上面是execute方法，发现execute其实内部调用的executeOnExecutor方法，调用executeOnExecutor方法传递了两个参数，这里第一个传递了一个默认的执行器，关于这个sDefaultExecutor我们再来讲，我们现在来看AsyncTask的流程设计。

我们来看这个executeOnExecutor方法,这里将不重要的代码略去了，其实这里面的逻辑比较清晰。
```
 public final AsyncTask<Params, Progress, Result> executeOnExecutor(Executor exec,
            Params... params) {
        .....
        .....

        mStatus = Status.RUNNING;
        ////////////////////////////////
        // 第一步：在主线程中执行准备操作////
        onPreExecute();
        // 第二步：把params参数赋值给mWorker
        mWorker.mParams = params;
        // 第三步：用线程池执行mFuture
        exec.execute(mFuture);
        ///////////////////////
        return this;
    }
```
第一步，onPreExecute与上面的介绍一样，进行准备工作，这个就没有必要分析，如果我们没有重写，就不会做相关的准备。我们主要第二步和第三步，第二步，把params参数赋值给mWorker，params是execute中传递过来的参数，同时也是泛型中第一个参数，将params赋值给mWorker，那么mWorker是什么呢？

>#### AsyncTask的构造方法
我们继续看源码:主意到mWorker是在AsyncTask的构造方法中创建的。

```
public AsyncTask() {
        mWorker = new WorkerRunnable<Params, Result>() {
            public Result call() throws Exception {
               .....
            }
        };

        mFuture = new FutureTask<Result>(mWorker) {
            @Override
            protected void done() {
               .....
            }
        };
    }
```
首先，mWorker在构造方法中创建，它是一个匿名内部类，那WorkerRunnable是个什么东西呢？
```
 private static abstract class WorkerRunnable<Params, Result> implements Callable<Result> {
        Params[] mParams;
    }
```

我们发现WorkerRunnable其实就是一个Callable，同时在execute方法中mFuture也在这里创建了出来，这里会将mWorker传递到FutureTask中去，那么将mFuture传进去做了什么什么操作呢？

```
 public FutureTask(Callable<V> callable) {
        if (callable == null)
            throw new NullPointerException();
        //////这里将mWorker传递进来，其实就是callable///
        /////////////
        this.callable = callable;
        this.state = NEW;       // ensure visibility of callable
    }
```
FutureTask是java.util.concurrent，FutureTask继承了Future，通过 Future 接口，可以尝试取消尚未完成的任务，查询任务已经完成还是取消了，以及提取（或等待）任务的结果值。

在AsyncTask的构造方法中，将mWorker传进来，即将callable传进来，因为mWorker就是callable。

这样在上面的executeOnExecutor中的第三步中，==exec.execute(mFuture)== 用线程池来执行mFuture，其实就是执行mFuture中的run方法，我们来看FutureTask中的run方法：
>#### FutureTask中的run方法

```
 public void run() {
        if (state != NEW ||
            !UNSAFE.compareAndSwapObject(this, runnerOffset,
                                         null, Thread.currentThread()))
            return;
        try {
            Callable<V> c = callable;
            if (c != null && state == NEW) {
                V result;
                boolean ran;
                try {
                   //① 调用callable中的call方法，其实就是mWorker中的call方法
                   //并且将结果赋值给result
                    result = c.call();
                    ran = true;
                } catch (Throwable ex) {
                    result = null;
                    ran = false;
                    setException(ex);
                }
                if (ran)
                    //② 调用自己内部的set方法设置结果
                    set(result);
            }
        } finally {
            // runner must be non-null until state is settled to
            // prevent concurrent calls to run()
            runner = null;
            // state must be re-read after nulling runner to prevent
            // leaked interrupts
            int s = state;
            if (s >= INTERRUPTING)
                handlePossibleCancellationInterrupt(s);
        }
    }
```

在FutureTask中的run方法，我们需要关注两个地方，第一个，就是上面代码片段的①处，这里调用了mWorker中的call方法，这样我们再回头来看mWorker中的call方法。
```
mWorker = new WorkerRunnable<Params, Result>() {
            public Result call() throws Exception {
                mTaskInvoked.set(true);

                Process.setThreadPriority(Process.THREAD_PRIORITY_BACKGROUND);
                //noinspection unchecked
                return postResult(doInBackground(mParams));
            }
        };
```
```
 private Result postResult(Result result) {
        @SuppressWarnings("unchecked")
        Message message = sHandler.obtainMessage(MESSAGE_POST_RESULT,
                new AsyncTaskResult<Result>(this, result));
        message.sendToTarget();
        return result;
    }
```
在mWorker中call方法中主要就是执行耗时操作，正是doInBackground方法，并且将执行的结果result返回回去，用postResult对doInBackground进行包裹则是为了运用Handler机制来更新UI。
接下来我们看FutureTask中run方法中的②处，调用了FutureTask自己的set方法。

```
protected void set(V v) {
        if (UNSAFE.compareAndSwapInt(this, stateOffset, NEW, COMPLETING)) {
            outcome = v;
            UNSAFE.putOrderedInt(this, stateOffset, NORMAL); // final state
            finishCompletion();
        }
    }
```
```
 private void finishCompletion() {
        // assert state > COMPLETING;
        for (WaitNode q; (q = waiters) != null;) {
            if (UNSAFE.compareAndSwapObject(this, waitersOffset, q, null)) {
                for (;;) {
                    Thread t = q.thread;
                    if (t != null) {
                        q.thread = null;
                        LockSupport.unpark(t);
                    }
                    WaitNode next = q.next;
                    if (next == null)
                        break;
                    q.next = null; // unlink to help gc
                    q = next;
                }
                break;
            }
        }
        //① 调用了FutureTask中的done方法
        done();

        callable = null;        // to reduce footprint
    }
```
由set方法，调用finishCompletion，主要看finishCompletion的逻辑，我们只关注finishCompletion代码的①处，这里调用了done方法，
这样我们来看done方法中的逻辑。

```
mFuture = new FutureTask<Result>(mWorker) {
            @Override
            protected void done() {
                try {
                    postResultIfNotInvoked(get());
                } catch (InterruptedException e) {
                    android.util.Log.w(LOG_TAG, e);
                } catch (ExecutionException e) {
                    throw new RuntimeException("An error occured while executing doInBackground()",
                            e.getCause());
                } catch (CancellationException e) {
                    postResultIfNotInvoked(null);
                }
            }
        };
```
在Future中调用了postResultIfNotInvoked，其实这里将这段处理逻辑抽取到方法中去了，在android2.3即以前的源码都是没有抽取的，这也是使得现在的逻辑更加清晰。
==java.util.concurrent.atomic.AtomicBoolean （ 在这个Boolean值的变化的时候不允许在之间插入，保持操作的原子性==）
由于在mWorker中的call和在mFuture的done方法都会调用postResult来更新UI，由于是线程操作，不能保证先后顺序，所以需要使用AtomicBoolean来保持操作的原子性。其实在2.3上的代码不是这样处理的，2.3上将更新UI的操作都放在mFuture中的done方法中。

```
private void postResultIfNotInvoked(Result result) {
        final boolean wasTaskInvoked = mTaskInvoked.get();
        if (!wasTaskInvoked) {
            postResult(result);
        }
    }
```
最后，我们来看postResult方法。

```
  private Result postResult(Result result) {
        @SuppressWarnings("unchecked")
        Message message = sHandler.obtainMessage(MESSAGE_POST_RESULT,
                new AsyncTaskResult<Result>(this, result));
        message.sendToTarget();
        return result;
    }
```
这里就是我们非常熟悉的代码了，使用Message发送消息给Handler来更新UI。
在AsyncTask中定义了一个InternalHandler，如果耗时操作执行完毕，就会执行finish(result.mData[0])，如果结果正在执行，则会onProgressUpdate来更新进度，这个onProgressUpdate正是我们前面说到的需要实现的更新进度的方法。
```
private static class InternalHandler extends Handler {
        @SuppressWarnings({"unchecked", "RawUseOfParameterizedType"})
        @Override
        public void handleMessage(Message msg) {
            AsyncTaskResult result = (AsyncTaskResult) msg.obj;
            switch (msg.what) {
                case MESSAGE_POST_RESULT:
                    // There is only one result
                    result.mTask.finish(result.mData[0]);
                    break;
                case MESSAGE_POST_PROGRESS:
                    result.mTask.onProgressUpdate(result.mData);
                    break;
            }
        }
    }
```
AsyncTaskResult类的只是一个封装
```
  @SuppressWarnings({"RawUseOfParameterizedType"})
    private static class AsyncTaskResult<Data> {
        final AsyncTask mTask;
        final Data[] mData;

        AsyncTaskResult(AsyncTask task, Data... data) {
            mTask = task;
            mData = data;
        }
    }
```

```
private void finish(Result result) {
        if (isCancelled()) {
            onCancelled(result);
        } else {
            //////耗时操作执行完毕，更新UI///////////
            //onPostExecute运行在主线程
            onPostExecute(result);
            //////////////////////////////////////////
        }
        mStatus = Status.FINISHED;
    }
```
##############################################################

这样我们关于AsyncTask的流程终于走通了，为什么onPreExecute和onPostExecute运行在主线程，而doingBackGround为什么运行在子线程中，这个逻辑是不是就变得清晰了。上面贴了好多代码，一直都是在分析代码的意思，至于关于设计的思想感觉现在的自己还体悟不够。

4. AsyncTask在上面遗留的问题
关于AsyncTask中executeOnExecutor中的sDefaultExecutor

项目中问题场景：

操作步骤>>>>>>>>>>>
- 设置安全中，选择指纹。
- 解锁方式选择图案。
- 在选择您的图案界面，点击确定，需要三到五秒才能跳转到下一界面。

问题分析：在设置解锁方式为为图案时，第二次绘制图案后，Settings源码中使用了AsyncTask来将一些比较耗时的验证操作放在子线程中去处理(参看下面的部分代码)，由于android原生设计是使用
AsyncTask中的一个参数的方法，一个参数的方法采用的是默认的执行器，即串行执行器

==frameworks/base/core/java/com/android/internal/widget/LockPatternChecker.java==

```
public static AsyncTask<?, ?, ?> verifyPattern(final LockPatternUtils utils,
            final List<LockPatternView.Cell> pattern,
            final long challenge,
            final int userId,
            final OnVerifyCallback callback) {
        AsyncTask<Void, Void, byte[]> task = new AsyncTask<Void, Void, byte[]>() {
            private int mThrottleTimeout;
            @Override
            protected byte[] doInBackground(Void... args) {
                try {
                    return utils.verifyPattern(pattern, challenge, userId);
                } catch (RequestThrottledException ex) {
                    mThrottleTimeout = ex.getTimeoutMs();
                    return null;
                }
            }
            @Override
            protected void onPostExecute(byte[] result) {
                callback.onVerified(result, mThrottleTimeout);
            }
        };
        ////////////默认使用的串行的执行器//////////////
        task.execute();
        ///////////////////////////////////////////////
        return task;
    }
```


```
public static final Executor SERIAL_EXECUTOR = new SerialExecutor();
private static volatile Executor sDefaultExecutor = SERIAL_EXECUTOR;
```

```
private static class SerialExecutor implements Executor {
        final ArrayDeque<Runnable> mTasks = new ArrayDeque<Runnable>();
        Runnable mActive;

        public synchronized void execute(final Runnable r) {
            mTasks.offer(new Runnable() {
                public void run() {
                    try {
                        r.run();
                    } finally {
                        scheduleNext();
                    }
                }
            });
            if (mActive == null) {
                scheduleNext();
            }
        }

        protected synchronized void scheduleNext() {
            if ((mActive = mTasks.poll()) != null) {
                THREAD_POOL_EXECUTOR.execute(mActive);
            }
        }
    }
```

> ##### 我们发现SerialExecutor即是串行执行器，它的作用是保证任务执行的顺序，也就是它可以保证提交的任务确实是按照先后顺序执行的。它的内部有一个队列用来保存所提交的任务，保证当前只运行一个，这样就可以保证任务是完全按照顺序执行的。如果发现异步任务还未执行，可能被我们发现SerialExecutor即是串行执行器顺序的使用线程执行。因为应用中可能还有其他地方使用AsyncTask，所以到我们的AsyncTask也许会等待到其他任务都完成时才得以执行而不是调用executor()之后马上执行。

如果executeOnExecutor(AsyncTask.THREAD_POOL_EXECUTOR,params)则不一样。
```
 /**
     * An {@link Executor} that can be used to execute tasks in parallel.
     */
    public static final Executor THREAD_POOL_EXECUTOR
            = new ThreadPoolExecutor(CORE_POOL_SIZE, MAXIMUM_POOL_SIZE, KEEP_ALIVE,
                    TimeUnit.SECONDS, sPoolWorkQueue, sThreadFactory);
```
> ##### API中的解释：能够并行的执行任务。THREAD_POOL_EXECUTOR是一个数量为corePoolSize的线程池，具体线程池的数量是依据CPU的核心来设置的，如果超过这个数量的线程个数就需要等待

SerialExecutor是按顺序执行，THREAD_POOL_EXECUTOR则一定程度上能保证并行执行。

以上就是关于AsyncTask的全部内容，希望能对你有些帮助，贴了好多源码，如果想真正弄清楚，还是得自己去阅读阅读源码，整理这个也不容易，前前后后花了大概一个星期。
最后是AsyncTask的时序图，画的不太好，凑合看吧，O(∩_∩)O哈哈~
![AsyncTask时序图](https://raw.githubusercontent.com/JasonGaoH/Images/master/AsyncTask.png)
 