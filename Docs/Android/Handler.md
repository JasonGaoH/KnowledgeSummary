Handler如何在handleMessage方法拦截之前发出的message

``` java

/**
* Handle system messages here.
*/
public void dispatchMessage(Message msg) {
//首先检查msg是否设置了回调
    if (msg.callback != null) {
        handleCallback(msg);
    } else {
        if (mCallback != null) {
        //重点在这里，callback的handleMessage()方法的返回值决定了是否拦截消息
        //重点在这里，callback的handleMessage()方法的返回值决定了是否拦截消息
        //重点在这里，callback的handleMessage()方法的返回值决定了是否拦截消息
            if (mCallback.handleMessage(msg)) {
                return;
            }
        }
        handleMessage(msg);
    }
}

```