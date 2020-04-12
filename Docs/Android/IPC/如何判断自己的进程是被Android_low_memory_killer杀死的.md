在 Android 系统中，Process 的生命周期是有系统控制的，当用户通过 back 键关掉 程序时，进程依然存在，其依然占用一定的内存。随着系统运行时间越长，用户打开的程序越多，整个系统占用的内存越多，Android 会定时执行一次检查，杀死一些进程，进而释放一些可用的内存空间，android 系统是通过 low memory killer 机制来实现的。

Android 的 low memory killer 是基于 linux 的 oom 规则改进而来的，OMS 通过一些比较复杂的评分机制，对每个进程进行打分，然后根据分数高低和系统配置的一些参数来决定 kill 哪些进程。

几个主要概念:

1. Process Importance:ActivityManager.RunningAppPrcessInfo 中定义了不同种类 App 的 Importance
2. oom_adj:ProcessList 中定义了不同类型进程的 adj。Low memory killer 主要是通过进程的 oom_adj 来判定进程的重要程度。oom_adj 的大小和进程的类型以及进程被调度的次序有关。
3. 系统目录/sys/module/lowmemorykiller/parameters/adj 中定义了当前系统配置的 oom_adj，例如:0,1,2,7,14,15。这些值最初是在 init.rc 中配置的
4. 系统目录/sys/module/lowmemorykiller/parameters/minfree 中定义了当前系统配置 的 minfreee(单位为 4k，page 的大小)，例如:1536,2048,4096,5120,5632,6144
5. 以上两组 int 数组是一一对应，即当系统剩余内存小于某个 minfree 时， lowmemorykiller 会杀死对应 oom_adj 的进程
6. 每个进程当前的 oom_adj 是实时变化的，即进程处于不同的状态，其 oom_adj 会随时发生变化。其值存在系统目录/proc/<PID>/oom_adj 中
7. 系统初始进程 init 进程的 oom_adj 为-16，是最小的，根据 oom_adj 和 minfree 的 配置，该进程永远不会被 kill。其值是在 init.rc 中设定的:write /proc/1/oom_adj -16 8. 查看系统剩余的内存:cat /proc/meminfo
9. Low memory killer 的具体实现可参看:kernel/drivers/misc/lowmemorykiller.c 后台程序被 kill 有多种原因，目前发现的可以解释的有两种:
   1. 由 restartPackage 传入 packageName 导致该包里所有的进程被 kill。
   2. 由 lowmemorykiller 当在系统内存紧张时根据优先级选择一些进程 kill
   第一类目前还没有很好的办法来避免，目前只能做到自己的 taskkiller 不要通过 restartPackage 来 kill 自己包里的任何进程。
   更多的主要是由第二类造成的，那么如何判断是否是由第二类造成的呢?可以通过 如下方法来判断:
   1. adb shell
   2. cat /sys/module/lowmemorykiller/parameters/adj，该命令会打印出该系统配置的进
   程优先级值。例如:0,1,2,7,14,15
   1. cat /sys/module/lowmemorykiller/parameters/minfree，该命令会打印系统配置的针对不同 adj 的最小内存阈值。例如:1536,2048,4096,5120,5632,6144
   2. 根据 ps 命令找到目标进程对应的 pid，比如 com.android.contacts 进程
   3. cat /proc/桌面进程 pid/oom_adj，会得到步骤 2 中输出的某个值比如 2，根据第二步中该值出现的顺序，找到第三步中输出的相同数据的那个值 B，比如 2048
   4. cat /proc/meminfo，找到 MemFree: 这一行，后面会有一个值 A，例如 79932 kB
   5. 比较 B*4 和 A 的大小，如果 A<B*4 或者基本接近，则这类是有 lowmemorykiller 即内存不足杀死的。