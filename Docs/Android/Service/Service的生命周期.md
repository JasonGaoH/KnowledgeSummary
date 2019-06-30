Service是运行在主线程中的，即主线程。

- startService() 开启Service，调用者退出后Service仍让存在。
- bindService() 开启Service，调用者退出后Service也随即退出。

Service的生命周期

- 只是startService的情况下，onCreate() -> onStartCommand() -> onDestroy()
- 只是bindService的情况下，onCreate() -> onBind() -> onUnBind() -> onDestroy()
