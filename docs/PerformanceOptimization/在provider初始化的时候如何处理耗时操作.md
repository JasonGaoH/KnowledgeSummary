如果 Provider 在 onCreate()的过程中需要做一些耗时的操作，例如数据的初始化，清理数据库中的垃圾数 据，或者重建数据的索引等等，像这一类的操作可以放到一个后台线程顺序执行。

代码：

```java

private HandlerThread mBackgroundThread; private Handler mBackgroundHandler;

@Override
public boolean onCreate() {
    super.onCreate();
    mBackgroundThread = new HandlerThread("ProviderWorker", Process.THREAD_PRIORITY_BACKGROUND);
    mBackgroundThread.start();
    
    mBackgroundHandler = new Handler(mBackgroundThread.getLooper()) {
        @Override
        public void handleMessage(Message msg) {
            performBackgroundTask(msg.what, msg.obj); }
    };
    // 任务在后台线程顺序执行 
    scheduleBackgroundTask(BACKGROUND_TASK_INITIALIZE); 
    scheduleBackgroundTask(BACKGROUND_TASK_UPDATE_LOCALE);
    scheduleBackgroundTask(BACKGROUND_TASK_UPDATE_SEARCH_INDEX); 
    return true;
}

protected void scheduleBackgroundTask(int task) { 
    mBackgroundHandler.sendEmptyMessage(task);
}

// 在线程中处理耗时的操作
protected void performBackgroundTask(int task, Object arg) {
    switch (task) { 
        case BACKGROUND_TASK_INITIALIZE: {
                doInitialize();
                break; 
            }
        case BACKGROUND_TASK_UPDATE_LOCALE: { 
            doIUpdateLocale();
            break; 
        }
        case BACKGROUND_TASK_UPDATE_SEARCH_INDEX: { 
            doIUpdateSearchIndex();
            break; 
        }
    }
}
```

这样做的好处是把耗时操作放在后台线程处理，使得 provider 的 onCreate()不会执行很长时间，provider 初始化完成之后，开始处理 query/insert/update/delete 的操作。

假如 query/insert/update/delete 等数据库操作发生在后台的任务结束之后，这样做看起来是没有问题的， 但是很有可能 doInitialize()还没有结束，就会有进程来操作数据库。

接下来需要对我们的实现做一下改进， 来确保数据库初始化完成之前 query/insert/update/delete 等数据库 操作 不会被执行

假设在 doInitialize()之后，我们就准备好可以进行读操作，而在 doIUpdateSearchIndex() 之后我们才允许 写操作。

假如 query/insert/update/delete 等数据库操作发生在后台的任务结束之后，这样做看起来是没有问题的， 但是很有可能 doInitialize()还没有结束，就会有进程来操作数据库。

接下来需要对我们的实现做一下改进， 来确保数据库初始化完成之前 query/insert/update/delete 等数据库 操作 不会被执行。

假设在 doInitialize()之后，我们就准备好可以进行读操作，而在 doIUpdateSearchIndex() 之后我们才允许 写操作

```java

private HandlerThread mBackgroundThread; 
private Handler mBackgroundHandler;

private volatile CountDownLatch mReadAccessLatch; 
private volatile CountDownLatch mWriteAccessLatch; 
    @Override
    public boolean onCreate() {
    super.onCreate();
    // The provider is closed for business until fully initialized mReadAccessLatch = new CountDownLatch(1); mWriteAccessLatch = new CountDownLatch(1);
    mBackgroundThread = new HandlerThread("ProviderWorker", Process.THREAD_PRIORITY_BACKGROUND);
    mBackgroundThread.start();
    mBackgroundHandler = new Handler(mBackgroundThread.getLooper()) {
        @Override
            public void handleMessage(Message msg) { 
                performBackgroundTask(msg.what, msg.obj);
        } };
    scheduleBackgroundTask(BACKGROUND_TASK_INITIALIZE); 
    scheduleBackgroundTask(BACKGROUND_TASK_UPDATE_LOCALE); 
    scheduleBackgroundTask(BACKGROUND_TASK_UPDATE_SEARCH_INDEX); 
    scheduleBackgroundTask(BACKGROUND_TASK_OPEN_WRITE_ACCESS);
}

protected void scheduleBackgroundTask(int task) { 
    mBackgroundHandler.sendEmptyMessage(task);
}

// 在线程中处理耗时的操作
protected void performBackgroundTask(int task, Object arg) {
    switch (task) {
    case BACKGROUND_TASK_INITIALIZE: {
        doInitialize(); 
        mReadAccessLatch.countDown();
        mReadAccessLatch = null;
        break; 
    }
    case BACKGROUND_TASK_UPDATE_LOCALE: { 
        doIUpdateLocale();
        break; 
    }
    case BACKGROUND_TASK_UPDATE_SEARCH_INDEX: { 
        doIUpdateSearchIndex();
        break; 
    }
    case BACKGROUND_TASK_OPEN_WRITE_ACCESS: { 
        mWriteAccessLatch.countDown();
        mWriteAccessLatch = null;
        break; 
    }
}
```

我们新增加了两个计数器 mReadAccessLatch 和 mWriteAccessLatch，初始化值都为 1，doInitialize()结束 之后 mReadAccessLatch 赋值为 null，所有后台任务结束之后 mWriteAccessLatch 赋值为 null， 这样在 query/insert/update/delete 之前先判断一下计数器就可以知道是否可以执行该操作了。

```java
private void waitForAccess(CountDownLatch latch) { 
    if (latch == null) {
        return; 
    }
    while (true) { 
        try {
            latch.await();
            return;
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt(); }
        } 
    }

@Override
public Uri insert(Uri uri, ContentValues values) {
    waitForAccess(mWriteAccessLatch); return doInsert();
}

@Override
public int update(Uri uri, ContentValues values, String selection, String[] selectionArgs) {
    waitForAccess(mWriteAccessLatch);
    return doUpdate(); 
}

@Override
public int delete(Uri uri, String selection, String[] selectionArgs) {
    waitForAccess(mWriteAccessLatch); 
    return doDelete();
}

@Override
public Cursor query(Uri uri, String[] projection, String selection, String[] selectionArgs, String sortOrder, CancellationSignal cancellationSignal) {
    waitForAccess(mReadAccessLatch);
    return doQuery(); 
}
```

在数据库操作之前增加 waitForAccess()，这样在数据库初始化的过程中，会阻塞其他进程对数据库的写操 作，当初始化完成之后，所有被阻塞的操作再被执行。