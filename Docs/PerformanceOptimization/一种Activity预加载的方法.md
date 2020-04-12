一般某个 Activity 界面打开过程包括如下阶段:

1. 从发起打开命令“startActivity”到出现该界面所在窗口动画前 — 如果该阶段时间较长，给用户的感觉就是 点击后有一个等待时间，会觉得机器反应慢。
2. 窗口动画过程 — 这个过程可以自己定义，在 startActivity 时指定旧窗口和新窗口的动画，好的动画也会给用户很快的感觉。
3. 窗口动画完成后到数据加载完成和现实数据的界面展现出来 — 这个主要包括窗口中所有 view 的 inflate 时间，使用的所有资源的加载时间和具体数据的 load 时间等。

为了提升新界面打开速度，我们一般通过修改自己的代码来实现，比如减少 view 层次，延迟加载，缓存机制，多线程，预加载等方法。以上方法都是在自己的进程启动后来做的，但对如下这种情况不太适用:即用户打开某个 activity，但该 activity 所在的进程还没有创建和启动时，这时可能需要更长的等待时间。

这里介绍一种 activity 的预加载方法，并尝试避免以上问题，先贴出代码，后面再详细介绍：

```java
public class PreloadService extends IntentService {
    private final static String TAG = "PreloadService"; 
    public final static String EXTRA_PRELOAD = "preload"; 
    private static boolean sHasPreload = false;
    
    private static class LocalActivityRecord extends Binder { 
        LocalActivityRecord(String _id, Intent _intent) {
            id = _id;
            intent = _intent; 
        }
        final String id;
        Intent intent; 
    }

    public PreloadService() {
        super("PreloadService");
    }

    @Override
    protected void onHandleIntent(Intent intent) {
        Log.i(TAG, "PreloadService onHandleIntent..."); 
        if (sHasPreload) {
            return; 
        }
        ActivityThread activityThread = ActivityThread.currentActivityThread();
        if (activityThread == null) { //4.0 ~ 4.2 上 ActivityThread 的对象是 ThreadLocal 的，当前线程中 无法获取，所以只能在 mainThread 中获取
        final Looper mainLooper = Looper.getMainLooper(); 
        Handler mainHandler = new Handler(mainLooper); 
        mainHandler.post(new Runnable() {

            @Override
            public void run() {
                activityThread = ActivityThread.currentActivityThread();
                final ActivityThread finalActivityThread = activityThread; 
                new Thread(new Runnable() {
                    @Override
                    public void run() {
                        doPreload(finalActivityThread); }
                    }).start(); 
                }
            }); 
        } else {
            doPreload(activityThread); 
        }
        sHasPreload = true; 
    }

    private void doPreload(ActivityThread activityThread) {
        Intent activityIntent = null;
        Activity activity = null;
        if (XXXActivity.sShouldPreload) { //如果真正的启动正在运行，预加载就不需
            activityIntent = new Intent(getApplicationContext(), XXXActivity.class);
            activityIntent.putExtra(EXTRA_PRELOAD, true);
            String id = XXXActivity.class.getName();
            LocalActivityRecord activityRecord = new LocalActivityRecord(id, activityIntent); 
            activity = activityThread.startActivityNow(null, activityRecord.id, activityRecord.intent,
            activityThread.resolveActivityInfo(activityRecord.intent), activityRecord, null, null); 
            if (activity != null) {
                activityThread.performResumeActivity(activityRecord, true);
                activityThread.performDestroyActivity(activityRecord, true); 
            }
        } 
    }
}
```

启动预加载

```java

Intent intent = new Intent(getApplicationContext(), PreloadService.class); 
context.startService(intent);
```

基本原理

当 activity 所在进程还没有启动时，用户如果打开某个 activity，首先需要创建这个进程，要将 activity 使用 的 class，xml，resource 等从 apk 文件加载到内存，这个过程中有大量的 io，是比较耗时的。文中的预加 载方式可以很好的解决这个问题。

1. 首先将 PreloadService 放在与预加载 activity 相同的进程内
2. 在 PreloadService 中获取当前进程中唯一用来创建 activity 的底层的管理类:ActivityThread 的对象
3. ActivityThread 可以根据 intent 创建 intent 中制定的 activity 对象，Activity activity = activityThread.startActivityNow(...intent...)
4. 这样系统会创建该 activity，并且调用了 activity 的 onCreate，这样就会把 activity 的 contentView 加载进 来，如果在 onCreate 中调用了 fragmentManager.addFragment，这时 fragment 的 create 生命周期函数也 会调用
5. 因为能够使得 activity 加载的更彻底，再调用 activityThread.performResumeActivity(activityRecord, true)，这样会调用 activity 的 onResume，同样的 fragment 的也会调用
6. 为了能够使得预加载的 activity 可以被回收，需要调用 activityThread.performDestroyActivity(activityRecord, true)，该方法会将 activity 从 activityThread 中移除
7. 通过以上方法，在用户点击之前可以做预加载