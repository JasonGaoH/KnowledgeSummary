ANR(Application Not responding)。Android中，主线程(UI线程)如果在规定时内没有处理完相应工作，就会出现ANR。具体来说，ANR会在以下几种情况中出现:
1. 输入事件(按键和触摸事件)5s内没被处理
2. BroadcastReceiver的事件(onRecieve方法)在规定时间内没处理完(前台广播为10s，后台广播为60s)
3. service 前台20s后台200s未完成启动
4. ContentProvider的publish在10s内没进行完

[AndroidANR问题总结](https://www.jianshu.com/p/fa962a5fd939)