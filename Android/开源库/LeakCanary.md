# LeakCanary
[优秀博客](https://mp.weixin.qq.com/s/0bO5BZ4CMYJbRuY_xf_osw)

LeakCanary内存泄漏检测机制，大概是以下几个步骤：

1. 利用 application.registerActivityLifecycleCallbacks(lifecycleCallbacks) 来监听整个生命周期内的 Activity onDestoryed 事件;

2. 当某个 Activity 被 destory 后，将它传给 RefWatcher 去做观测，观察后续会被正常回收；

3. RefWatcher 首先把 Activity 使用 KeyedWeakReference 引用起来，并使用一个 ReferenceQueue 来记录该 KeyedWeakReference 指向的对象是否已被回收；

4. AndroidWatchExecutor 会在 5s 后，开始检查这个弱引用内的 Activity 是否被正常回收。判断条件是：若 Activity 被正常回收，那么引用它的 KeyedWeakReference 会被自动放入 ReferenceQueue 中。

5. 判断方式是：先看 Activity 对应的 KeyedWeakReference 是否已经放入 ReferenceQueue 中；如果没有，则手动 GC：gcTrigger.runGc();；然后再一次判断 ReferenceQueue 是否已经含有对应的 KeyedWeakReference。若还未被回收，则认为可能发生内存泄漏。

6. 利用 HeapAnalyzer 对 dump 的内存情况进行分析并进一步确认，若确定发生泄漏，则利用 DisplayLeakService 发送通知。


对于最后一步的分析过程，详细如下：

1. 如果引用还是未被清除，把 heap 内存 dump 到一个 .hprof 文件中；

2. 在另外一个进程中的 HeapAnalyzerService 有一个 HeapAnalyzer 使用HAHA 解析这个文件；

3. 得益于唯一的 reference key, HeapAnalyzer 找到 KeyedWeakReference，定位内存泄露；

4. HeapAnalyzer 计算 到 GC roots 的最短强引用路径，并确定是否是泄露。如果是的话，建立导致泄露的引用链；

5. 引用链传递到 APP 进程中的 DisplayLeakService， 并以通知的形式展示出来。


LeakCanary 使用方式：

```
public class ExampleApplication extends Application {
  @Override public void onCreate() {
    super.onCreate();
    if (LeakCanary.isInAnalyzerProcess(this)) {
      // This process is dedicated to LeakCanary for heap analysis.
      // You should not init your app in this process.
      return;
    }
    
    // 初始化
    LeakCanary.install(this);
  }
}

dependencies {
 debugCompile 'com.squareup.leakcanary:leakcanary-android:1.5.1'
 releaseCompile 'com.squareup.leakcanary:leakcanary-android-no-op:1.5.1'
 testCompile 'com.squareup.leakcanary:leakcanary-android-no-op:1.5.1'
}

```

相关函数：

```
ActivityRefWatcher

// LeakCanary的初始化方法
public static RefWatcher install(Application application) {
    return refWatcher(application).listenerServiceClass(DisplayLeakService.class)
        .excludedRefs(AndroidExcludedRefs.createAppDefaults().build())
        .buildAndInstall();
  }
  

**AndroidRefWatcherBuilder.java**

  /**
   * Creates a {@link RefWatcher} instance and starts watching activity references 
   */
  public RefWatcher buildAndInstall() {
    RefWatcher refWatcher = build();
    if (refWatcher != DISABLED) {
      LeakCanary.enableDisplayLeakActivity(context);
      ActivityRefWatcher.install((Application) context, refWatcher);
    }
    return refWatcher;
  }
  

**ActivityRefWatcher.java **

public static void install(Application application, RefWatcher refWatcher) {
    new ActivityRefWatcher(application, refWatcher).watchActivities();
  }

	// 监听所有Activity生命周期
  private final Application.ActivityLifecycleCallbacks lifecycleCallbacks =
      new Application.ActivityLifecycleCallbacks() {
        @Override public void onActivityCreated(Activity activity, Bundle savedInstanceState) {
        }

        @Override public void onActivityStarted(Activity activity) {
        }

        @Override public void onActivityResumed(Activity activity) {
        }

        @Override public void onActivityPaused(Activity activity) {
        }

        @Override public void onActivityStopped(Activity activity) {
        }

        @Override public void onActivitySaveInstanceState(Activity activity, Bundle outState) {
        }

        @Override public void onActivityDestroyed(Activity activity) {
        
        // 实际上只监听onDestroy()
          ActivityRefWatcher.this.onActivityDestroyed(activity);
        }
      };
      

// 使用RefWatcher监控Activity
  void onActivityDestroyed(Activity activity) {
    refWatcher.watch(activity);
  }
```
很好理解，当一个Activity的onDestroy()执行后，该Activity应该为null。如果不为null，说明内存泄漏了。所以，只需要在onDestroy()将当前Activity丢给RefWatcher监听就好。

   
```
**RefWatcher.java**

public void watch(Object watchedReference, String referenceName) {
    if (this == DISABLED) {
      return;
    }
    checkNotNull(watchedReference, "watchedReference");
    checkNotNull(referenceName, "referenceName");
    final long watchStartNanoTime = System.nanoTime();
    
    // 每个被监控的Activity都对应生成一个Key
    String key = UUID.randomUUID().toString();
    
    // 将所有Activity的key存储下来
    retainedKeys.add(key);
    
    // 将传入的 Activity 包装成了一个 KeyedWeakReference
    final KeyedWeakReference reference =
        new KeyedWeakReference(watchedReference, key, referenceName, queue);

    ensureGoneAsync(watchStartNanoTime, reference);
  }


Retryable.Result ensureGone(final KeyedWeakReference reference, final long watchStartNanoTime) {
    long gcStartNanoTime = System.nanoTime();
    long watchDurationMs = NANOSECONDS.toMillis(gcStartNanoTime - watchStartNanoTime);

	// 把已被回收的对象的Key从retainedKeys中移除，即ReferenceQueue中的Reference。剩下的都是泄漏的Activity。
    removeWeaklyReachableReferences();

    if (debuggerControl.isDebuggerAttached()) {
      // The debugger can create false leaks.
      return RETRY;
    }
    
    // 判断某个reference的key是否仍在 retainedKeys 里，若不在，表示已回收，否则继续
    if (gone(reference)) {
      return DONE;
    }
    
    // 手动GC，回收所有走完onDestroy()的Activity
    gcTrigger.runGc();
    
    // 再次移除被GC回收的Activity
    removeWeaklyReachableReferences();
    
    // 如果该 reference 还在 retainedKeys 里，表示泄漏
    if (!gone(reference)) {
      long startDumpHeap = System.nanoTime();
      long gcDurationMs = NANOSECONDS.toMillis(startDumpHeap - gcStartNanoTime);

	  // 利用 heapDumper 把内存情况 dump 成文件
	  // 内部实现是用了JVM自带方法（Debug.dumpHprofData(filePath)）
      File heapDumpFile = heapDumper.dumpHeap();
      
      if (heapDumpFile == RETRY_LATER) {
        // Could not dump the heap.
        return RETRY;
      }
      
      long heapDumpDurationMs = NANOSECONDS.toMillis(System.nanoTime() - startDumpHeap);
      
      // 调用 heapdumpListener 进行内存分析，进一步确认是否发生内存泄漏。如果确认发生内存泄漏，调用 DisplayLeakService 发送通知。
      
      heapdumpListener.analyze(
          new HeapDump(heapDumpFile, reference.key, reference.name, excludedRefs, watchDurationMs,
              gcDurationMs, heapDumpDurationMs));
    }
    return DONE;
  }
```
解释下引用队列(ReferenceQueue)的作用：

对Activity使用弱引用，WeakReference<Activity> reference = new WeakReference(activity)。当GC扫描到该弱引用时，该Activity就会被回收，reference会被放入内部的ReferenceQueue中。

也就是说，从ReferenceQueue中取出来的所有Reference都是被回收的。

回到上面代码，所有被监控的Activity对应的Key，都被存储到retainedKeys中。只要把ReferenceQueue中所有的 reference 取出来，并把对应 retainedKeys 里的 key 移除，剩下的 key 对应的对象都没有被回收。

后续执行流程方法注释中已写出。

```
ServiceHeapDumpListener.java

@Override 
public void analyze(HeapDump heapDump) {
    checkNotNull(heapDump, "heapDump");
    
    // 在监听器中使用HeapAnalyzerService(独立线程，继承IntentService)分析内存泄漏情况
    HeapAnalyzerService.runAnalysis(context, heapDump, listenerServiceClass);
  }
```

开启独立进程，分析内存泄漏：

```
/**
 * This service runs in a separate process to avoid slowing down the app process or  making it run out of memory.
 */
 
public final class HeapAnalyzerService extends IntentService {

  private static final String LISTENER_CLASS_EXTRA = "listener_class_extra";
  private static final String HEAPDUMP_EXTRA = "heapdump_extra";

  public static void runAnalysis(Context context, HeapDump heapDump,
      Class<? extends AbstractAnalysisResultService> listenerServiceClass) {
    Intent intent = new Intent(context, HeapAnalyzerService.class);
    intent.putExtra(LISTENER_CLASS_EXTRA, listenerServiceClass.getName());
    intent.putExtra(HEAPDUMP_EXTRA, heapDump);
    context.startService(intent);
  }


  @Override protected void onHandleIntent(Intent intent) {
    if (intent == null) {
      CanaryLog.d("HeapAnalyzerService received a null intent, ignoring.");
      return;
    }
    
    String listenerClassName = intent.getStringExtra(LISTENER_CLASS_EXTRA);
    HeapDump heapDump = (HeapDump) intent.getSerializableExtra(HEAPDUMP_EXTRA);

    HeapAnalyzer heapAnalyzer = new HeapAnalyzer(heapDump.excludedRefs);

	// 内部使用 HaHa 解析hprof文件
    AnalysisResult result = heapAnalyzer.checkForLeak(heapDump.heapDumpFile, heapDump.referenceKey);
    
    // 使用DisplayLeakService展示引用链。下面会讲到参数 listenerClassName 的作用
    AbstractAnalysisResultService.sendResultToListener(this, listenerClassName, heapDump, result);
  }
}

AndroidManifest.xml

 <service
        android:name=".internal.HeapAnalyzerService"
        android:process=":leakcanary" // Service处于应用私有的独立进程中
        android:enabled="false"
        />

```

先了解下如何让Service处于独立的进程：

```
在AndroidManifest中声明Service
1. 当android:process=".servicePublic"，就是为此服务开启了一个全局的独立进程。
2. 当android:process=":servicePrivate"，就是为此应用开启了一个私有的独立进程。
```

所以，从HeapAnalyzerService的注册方式，就知道它处于独立进程。


使用HeapAnalyzer计算泄漏的对象到GC roots 的最短强引用路径。

```
HeapAnalyzer.java 

/**
   * Searches the heap dump for a {@link KeyedWeakReference} instance with the corresponding key,
   * and then computes the shortest strong reference path from that instance to the GC roots.
   */
   
  public AnalysisResult checkForLeak(File heapDumpFile, String referenceKey) {
    long analysisStartNanoTime = System.nanoTime();

    if (!heapDumpFile.exists()) {
      Exception exception = new IllegalArgumentException("File does not exist: " + heapDumpFile);
      return failure(exception, since(analysisStartNanoTime));
    }

    try {
	  // 使用 HAHA 解析dump文件
      HprofBuffer buffer = new MemoryMappedFileBuffer(heapDumpFile);
      HprofParser parser = new HprofParser(buffer);
      Snapshot snapshot = parser.parse();
      deduplicateGcRoots(snapshot);

      Instance leakingRef = findLeakingReference(referenceKey, snapshot);

      // False alarm, weak reference was cleared in between key check and heap dump.
      if (leakingRef == null) {
        return noLeak(since(analysisStartNanoTime));
      }

	  // 获取泄漏的对象到GC Roots 的最短强引用路径
      return findLeakTrace(analysisStartNanoTime, snapshot, leakingRef);
    } catch (Throwable e) {
      return failure(e, since(analysisStartNanoTime));
    }
  }
  
private AnalysisResult findLeakTrace(long analysisStartNanoTime, Snapshot snapshot,
      Instance leakingRef) {

    ShortestPathFinder pathFinder = new ShortestPathFinder(excludedRefs);
    ShortestPathFinder.Result result = pathFinder.findPath(snapshot, leakingRef);

    // False alarm, no strong reference path to GC Roots.
    if (result.leakingNode == null) {
      return noLeak(since(analysisStartNanoTime));
    }

    LeakTrace leakTrace = buildLeakTrace(result.leakingNode);

    String className = leakingRef.getClassObj().getClassName();

    // Side effect: computes retained size.
    snapshot.computeDominators();

    Instance leakingInstance = result.leakingNode.instance;

    long retainedSize = leakingInstance.getTotalRetainedSize();

    // TODO: check O sources and see what happened to android.graphics.Bitmap.mBuffer
    if (SDK_INT <= N_MR1) {
      retainedSize += computeIgnoredBitmapRetainedSize(snapshot, leakingInstance);
    }

    return leakDetected(result.excludingKnownLeaks, className, leakTrace, retainedSize,
        since(analysisStartNanoTime));
  }
  
```
其中，LeakTrace的官方解释：（最短强引用路径的链式结构）

A chain of references that constitute the shortest strong reference path from a leaking instance to the GC roots. Fixing the leak usually means breaking one of the references in that chain.


使用DisplayLeakService展示引用链：

```
public abstract class AbstractAnalysisResultService extends IntentService {

  public static void sendResultToListener(Context context, String listenerServiceClassName,
      HeapDump heapDump, AnalysisResult result) {
    Class<?> listenerServiceClass;
    try {
      listenerServiceClass = Class.forName(listenerServiceClassName);
    } catch (ClassNotFoundException e) {
      throw new RuntimeException(e);
    }
    
    // 上面说到listenerClassName，就是被启动的Service。那listenerClassName值是什么呢？
    
    Intent intent = new Intent(context, listenerServiceClass);
    intent.putExtra(HEAP_DUMP_EXTRA, heapDump);
    intent.putExtra(RESULT_EXTRA, result);
    context.startService(intent);
  }
  
}
```

listenerClassName 是在 LeakCanary.install() 中定义的，就是DisplayLeakService：

```
LeakCanary.java 

public static RefWatcher install(Application application) {
    return refWatcher(application).listenerServiceClass(DisplayLeakService.class)
        .excludedRefs(AndroidExcludedRefs.createAppDefaults().build())
        .buildAndInstall();
  }
```

至此，LeakCanary就解释的差不多了。工作大概流程在文初已经给出。