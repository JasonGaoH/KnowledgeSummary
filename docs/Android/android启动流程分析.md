### android启动

> #### 当引导程序启动Linux内核后，会加载各种驱动和数据结构，当有了驱动以后，开始启动Android系统同时会加载用户级别的第一个进程init（system\core\init\init.cpp）代码如下：

```
int main(int argc, char** argv) {

    .....
    //创建文件夹，挂载
    // Get the basic filesystem setup we need put together in the initramdisk
    // on / and then we'll let the rc file figure out the rest.
    if (is_first_stage) {
         mount("tmpfs", "/dev", "tmpfs", MS_NOSUID, "mode=0755");
         mkdir("/dev/pts", 0755);
         mkdir("/dev/socket", 0755);
         mount("devpts", "/dev/pts", "devpts", 0, NULL);
         #define MAKE_STR(x) __STRING(x)
         mount("proc", "/proc", "proc", 0, "hidepid=2,gid=" MAKE_STR(AID_READPROC));
         mount("sysfs", "/sys", "sysfs", 0, NULL);
    }
    .....

    //打印日志，设置log的级别
    klog_init();
    klog_set_level(KLOG_NOTICE_LEVEL);
    .....
    
    Parser& parser = Parser::GetInstance();
    parser.AddSectionParser("service",std::make_unique<ServiceParser>());
    parser.AddSectionParser("on", std::make_unique<ActionParser>());
    parser.AddSectionParser("import", std::make_unique<ImportParser>());
    // 加载init.rc配置文件
    parser.ParseConfig("/init.rc");
}
```
> #### 加载init.rc文件，会启动一个Zygote进程，此进程是Android系统的一个母进程，用来启动Android的其他服务进程，代码：
从android L开始，在 /system/core/rootdir 目录中有 4 个 zygote 相关的启动脚本如下图：

![image](https://raw.githubusercontent.com/JasonGaoH/Images/master/init_rc_7.0.png)

在init.rc文件中，有如下代码：
```
import /init.environ.rc
import /init.usb.rc
import /init.${ro.hardware}.rc
import /init.usb.configfs.rc
import /init.${ro.zygote}.rc
```
注意到上面的代码import /init.${ro.zygote}.rc，这里会读取ro.zygote这个属性，导入相应的init.rc文件。
ro.zygote的属性可为：zygote32、zygote64、zygote32_64、zygote64_32。

对于这个属性的解释如下。
```
init.zygote32.rc：zygote 进程对应的执行程序是 app_process (纯 32bit 模式)
init.zygote64.rc：zygote 进程对应的执行程序是 app_process64 (纯 64bit 模式)
init.zygote32_64.rc：启动两个 zygote 进程 (名为 zygote 和 zygote_secondary)，对应的执行程序分别是 app_process32 (主模式)、app_process64。
init.zygote64_32.rc：启动两个 zygote 进程 (名为 zygote 和 zygote_secondary)，对应的执行程序分别是 app_process64 (主模式)、app_process32
```
主流的机型属性都为zygote64_32，我们来看看init.zygote64_32.rc文件：
```
service zygote /system/bin/app_process64 -Xzygote /system/bin --zygote --start-system-server --socket-name=zygote
   class main
   socket zygote stream 660 root system
   onrestart write /sys/android_power/request_state wake
   onrestart write /sys/power/state on
   onrestart restart audioserver
   onrestart restart cameraserver
   onrestart restart media
   onrestart restart netd
    writepid /dev/cpuset/foreground/tasks

service zygote_secondary /system/bin/app_process32 -Xzygote /system/bin --zygote --socket-name=zygote_secondary
    class main
    socket zygote_secondary stream 660 root system
    onrestart restart zygote
    writepid /dev/cpuset/foreground/tasks
```
在 linux 系统中，service 通常是一种被称为守护进程 (daemon) 的程序。它通常在系统启动时启动，并一直运行于后台，直到系统关闭时终止。

这里会以service方式来启动zygote进程，app_process的代码位于/frameworks/base/cmds/app_process/路径下，该路径下有一个文件app_main.cpp,入口是main函数。

> #### 接下来看app_main.cpp函数，这里实现从c++代码调到java代码：

```
int main(int argc, char* const argv[])
{
    .....
    //android运行时环境
    AppRuntime runtime(argv[0], computeArgBlockSize(argc, argv));
    // Process command line arguments
    // ignore argv[0]
    argc--;
    argv++;

    .....
    if (zygote) {
        //启动java代码
        runtime.start("com.android.internal.os.ZygoteInit", args, zygote);
    } else if (className) {
        runtime.start("com.android.internal.os.RuntimeInit", args, zygote);
    } else {
        fprintf(stderr, "Error: no class name or --zygote supplied.\n");
        app_usage();
        LOG_ALWAYS_FATAL("app_process: no class name or --zygote supplied.");
        return 10;
    }
}
```
> #### ZygoteInit.java 代码：
/frameworks/base/core/java/com/android/internal/os/ZygoteInit.java
```
    public static void main(String argv[]) {
        .....
        try {
            .....
            boolean startSystemServer = false;
            String socketName = "zygote";
            String abiList = null;
            for (int i = 1; i < argv.length; i++) {
                if ("start-system-server".equals(argv[i])) {
                    startSystemServer = true;
                } else if (argv[i].startsWith(ABI_LIST_ARG)) {
                    abiList = argv[i].substring(ABI_LIST_ARG.length());
                } else if (argv[i].startsWith(SOCKET_NAME_ARG)) {
                    socketName = argv[i].substring(SOCKET_NAME_ARG.length());
                } else {
                    throw new RuntimeException("Unknown command line argument: " + argv[i]);
                }
            }

            .....
            //预加载android依赖的文件
            preload();
			.....

            if (startSystemServer) {
                //启动系统服务
                startSystemServer(abiList, socketName);
            }

			.....
        } catch (MethodAndArgsCaller caller) {
            caller.run();
        } catch (Throwable ex) {
            Log.e(TAG, "Zygote died with exception", ex);
            closeServerSocket();
            throw ex;
        }
    }
```
看这个类的main方法，最上面根据传入的参数判断是否将startSystemServer这个标记设为true，接着预加载android依赖的文件，最后根据上面设置的标记来判断是否能启动系统服务。

接着看ZygoteInit.java中main方法在最后调用startSystemServer方法具体内容
```
    private static boolean startSystemServer(String abiList, String socketName)
            throws MethodAndArgsCaller, RuntimeException {
		.....
        /* Hardcoded command line to start the system server */
        String args[] = {
            "--setuid=1000",
            "--setgid=1000",
            "--setgroups=1001,1002,1003,1004,1005,1006,1007,1008,1009,1010,1018,1021,1032,3001,3002,3003,3006,3007,3009,3010",
            "--capabilities=" + capabilities + "," + capabilities,
            "--nice-name=system_server",
            "--runtime-args",
            "com.android.server.SystemServer",
        };
		.....
        try {
            parsedArgs = new ZygoteConnection.Arguments(args);
            ZygoteConnection.applyDebuggerSystemProperty(parsedArgs);
            ZygoteConnection.applyInvokeWithSystemProperty(parsedArgs);

            //Zygote进程开始fork子进程，启动SystemServer
            /* Request to fork the system server process */
            pid = Zygote.forkSystemServer(
                    parsedArgs.uid, parsedArgs.gid,
                    parsedArgs.gids,
                    parsedArgs.debugFlags,
                    null,
                    parsedArgs.permittedCapabilities,
                    parsedArgs.effectiveCapabilities);
        } catch (IllegalArgumentException ex) {
            throw new RuntimeException(ex);
        }
		.....
    }
```
> #### SystemServer.java 代码
/frameworks/base/services/java/com/android/server/SystemServer.java
```
      /**
    * The main entry point from zygote.
    */
      public static void main(String[] args) {
        new SystemServer().run();
      }

    private void run() {
        ......
        // Start services.
        try {
            Trace.traceBegin(Trace.TRACE_TAG_SYSTEM_SERVER, "StartServices");
            startBootstrapServices();
            //启动核心Service
            startCoreServices();
            //启动其他Service
            startOtherServices();
        } catch (Throwable ex) {
            Slog.e("System", "******************************************");
            Slog.e("System", "************ Failure starting system services", ex);
            throw ex;
        } finally {
            Trace.traceEnd(Trace.TRACE_TAG_SYSTEM_SERVER);
        }

        // Loop forever.
        Looper.loop();
        throw new RuntimeException("Main thread loop unexpectedly exited");
    }
```
> #### 启动SystemServer.java的startOtherServices方法

```
/**
 * Starts a miscellaneous grab bag of stuff that has yet to be refactored
 * and organized.
 */
private void startOtherServices() {
 .....
 // We now tell the activity manager it is okay to run third party
 // code.  It will call back into us once it has gotten to the state
 // where third party code can really run (but before it has actually
 // started launching the initial applications), for us to complete our
 // initialization.
 mActivityManagerService.systemReady(new Runnable() {
	 @Override
	 public void run() {
		 Slog.i(TAG, "Making services ready");
		 .....
	  }
	}
 .....
}
```
> #### ActivityManagerService.java中的mStackSupervisor.resumeFocusedStackTopActivityLocked()
```
   public void systemReady(final Runnable goingCallback) {
        .....  
       Slog.i(TAG, "System now ready");
       EventLog.writeEvent(EventLogTags.BOOT_PROGRESS_AMS_READY,
           SystemClock.uptimeMillis());
        .....
         
       synchronized (this) {
           // Only start up encryption-aware persistent apps; once user is
           // unlocked we'll come back around and start unaware apps
           startPersistentApps(PackageManager.MATCH_DIRECT_BOOT_AWARE);
			.....
	    //调用ActivityStackSupervisor中的resumeFocusedStackTopActivityLocked方法
           mStackSupervisor.resumeFocusedStackTopActivityLocked();
           mUserController.sendUserSwitchBroadcastsLocked(-1, currentUserId);
       }
   }
```
> #### ActivityStackSupervisor.java中的resumeFocusedStackTopActivityLocked方法
```
    boolean resumeFocusedStackTopActivityLocked() {
        return resumeFocusedStackTopActivityLocked(null, null, null);
    }

boolean resumeFocusedStackTopActivityLocked(
            ActivityStack targetStack, ActivityRecord target, ActivityOptions targetOptions) {
        if (targetStack != null && isFocusedStack(targetStack)) {
            return targetStack.resumeTopActivityUncheckedLocked(target, targetOptions);
        }
        final ActivityRecord r = mFocusedStack.topRunningActivityLocked();
        if (r == null || r.state != RESUMED) {
           //调用ActivityStack中的resumeTopActivityUncheckedLocked来保证栈中的顶层Activity被重新启动
            mFocusedStack.resumeTopActivityUncheckedLocked(null, null);
        }
        return false;
    }
```
> #### ActivityStack.java中的resumeTopActivityUncheckedLocked方法
```
   boolean resumeTopActivityUncheckedLocked(ActivityRecord prev, ActivityOptions options) {
       
       ....

       boolean result = false;
       try {
           // Protect against recursion.
           mStackSupervisor.inResumeTopActivity = true;
           if (mService.mLockScreenShown == ActivityManagerService.LOCK_SCREEN_LEAVING) {
               mService.mLockScreenShown = ActivityManagerService.LOCK_SCREEN_HIDDEN;
               mService.updateSleepIfNeededLocked();
           }
           result = resumeTopActivityInnerLocked(prev, options);
       } finally {
           mStackSupervisor.inResumeTopActivity = false;
       }
       return result;
   }
```
```
   private boolean resumeTopActivityInnerLocked(ActivityRecord prev, ActivityOptions options) {
        if (DEBUG_LOCKSCREEN) mService.logLockScreen("");
        if (!mService.mBooting && !mService.mBooted) {
           // Not ready yet!
           return false;
       } 
       .....
       if (next == null) {
            // There are no more activities!
            final String reason = "noMoreActivities";
            final int returnTaskType = prevTask == null || !prevTask.isOverHomeStack()
                    ? HOME_ACTIVITY_TYPE : prevTask.getTaskToReturnTo();
            if (!mFullscreen && adjustFocusToNextFocusableStackLocked(returnTaskType, reason)) {
                // Try to move focus to the next visible stack with a running activity if this
                // stack is not covering the entire screen.
                return mStackSupervisor.resumeFocusedStackTopActivityLocked(
                        mStackSupervisor.getFocusedStack(), prev, null);
            }

            // Let's just start up the Launcher...
            ActivityOptions.abort(options);
            if (DEBUG_STATES) Slog.d(TAG_STATES,
                    "resumeTopActivityLocked: No more activities go home");
            if (DEBUG_STACK) mStackSupervisor.validateTopActivitiesLocked();
            // Only resume home if on home display
            //调用ActivityStackSupervisor中的方法来恢复 Launcher所在的栈
            return isOnHomeDisplay() &&
                    mStackSupervisor.resumeHomeStackTask(returnTaskType, prev, reason);
        }
        .....
    }    
```
>##### ActivityStackSupervisor.java中resumeHomeStackTask方法
```
   boolean resumeHomeStackTask(int homeStackTaskType, ActivityRecord prev, String reason) {
        if (!mService.mBooting && !mService.mBooted) {
            // Not ready yet!
            return false;
        }

        .....

        mHomeStack.moveHomeStackTaskToTop(homeStackTaskType);
        ActivityRecord r = getHomeActivity();
        final String myReason = reason + " resumeHomeStackTask";

        // Only resume home activity if isn't finishing.
        if (r != null && !r.finishing) {
            mService.setFocusedActivityLocked(r, myReason);
            return resumeFocusedStackTopActivityLocked(mHomeStack, prev, null);
        }
		启动launcher应用的锁屏界面
        return mService.startHomeActivityLocked(mCurrentUser, myReason);
    }
```
>##### Android系统启动完成，打开了Launcher应用的Home界面。

![image](https://raw.githubusercontent.com/JasonGaoH/Images/master/android_init.png)