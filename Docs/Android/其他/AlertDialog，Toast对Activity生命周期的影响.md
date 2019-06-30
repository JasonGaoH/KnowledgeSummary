无论 Dialog 弹出覆盖页面，对 Activity 生命周期没有影响，只有再启动另外一个 Activity 的时候才会进入 onPause 状态，而不是想象中的被覆盖或者不可见.

同时通过 AlertDialog 源码或者 Toast 源码我们都可以发现它们实现的原理都是 windowmanager.addView();来添加的， 它们都是一个个 view ,因此不会对 activity 的生命周期有任何影响。

https://blog.csdn.net/cloud_castle/article/details/56011562