# EventBus


本篇博客涉及到的内容如下：

1. EventBus.register()过程
2. EventBus.post()过程
3. 注解Subscribe.priority作用
4. ConcurrentHashMap
5. CopyOnWriteArrayList
6. ThreadLocal


### 1. register过程：

1）通过反射查找是否被@Subscribe注解等过滤条件，找到EventBus所需要的onEvent系列回调方法；当然，过滤条件还包括方法只有一个参数、必须是public访问类型等；

2）定位到onEvent系列回调方法(即后面将到的Method)后，将Method、threadMode、eventType等存到SubscriberMethod；
    
3）将Subscriber(即注册EventBus的类，示例代码中的EventBusActivity)和自己的SubscriberMethod存到subscriptionsByEventType。post时就从subscriptionsByEventType获取对应Event的所有subscriber进行回调。


示例：

```
页面一：
public class EventBusActivity extends Activity {

    protected void onCreate(Bundle savedInstanceState) {
        EventBus.getDefault().register(this);
    }

    @Subscribe(threadMode = ThreadMode.MAIN)
    public void onStatusEvent(StatusEvent statusEvent) {
    }

    @Subscribe(threadMode = ThreadMode.BACKGROUND)
    public void onDescEvent(DescribeEvent event) {
    }
    
页面二：
public class PostEventActivity extends Activity {

    protected void onCreate(Bundle savedInstanceState) {
        EventBus.getDefault().register(this);
    }

    public void postEvent(View v) {
        EventBus.getDefault().post(new StatusEvent("idle"));
    }

    @Subscribe(threadMode = ThreadMode.MAIN, priority = 1)
    public void onStatusEvent(StatusEvent statusEvent) {
    }
}
```
其中，两个页面都注册了StatusEvent。


EventBus.java

```
核心数据结构 subscriptionsByEventType

// key: Event, 对应demo中的StatusEvent和DescribeEvent
// value: 注册了Event的所有subscriber。
// StatusEvent就有EventBusActivity和PostEventActivity两个subscriber，所以发射StatusEvent，两个Activity都会接收到
Map<Class<?>, CopyOnWriteArrayList<Subscription>> subscriptionsByEventType;


public void register(Object subscriber) { 
        Class<?> subscriberClass = subscriber.getClass();
        
		// 获取订阅者类里所有的注册回调方法，实现过程在SubscriberMethodFinder.java
		// 对应demo中的onStatusEvent()和onDescEvent()
		
        List<SubscriberMethod> subscriberMethods = subscriberMethodFinder.findSubscriberMethods(subscriberClass);
        
        synchronized (this) {
            for (SubscriberMethod subscriberMethod : subscriberMethods) {
                subscribe(subscriber, subscriberMethod);
            }
        }
    }
    
    
// 组装 subscriptionsByEventType
private void subscribe(Object subscriber, SubscriberMethod subscriberMethod) {
        Class<?> eventType = subscriberMethod.eventType;
        Subscription newSubscription = new Subscription(subscriber, subscriberMethod);
        CopyOnWriteArrayList<Subscription> subscriptions = subscriptionsByEventType.get(eventType);
        if (subscriptions == null) {
            subscriptions = new CopyOnWriteArrayList<>();
            subscriptionsByEventType.put(eventType, subscriptions);
        } else {
            ...
        }

        int size = subscriptions.size();
        for (int i = 0; i <= size; i++) {

        	// 根据method priority顺序填入，所以回调也会顺序执行。内容3的答案。
            if (i == size || subscriberMethod.priority > subscriptions.get(i).subscriberMethod.priority) {
                subscriptions.add(i, newSubscription);
                break;
            }
        }

        List<Class<?>> subscribedEvents = typesBySubscriber.get(subscriber);
        if (subscribedEvents == null) {
            subscribedEvents = new ArrayList<>();
            typesBySubscriber.put(subscriber, subscribedEvents);
        }
        subscribedEvents.add(eventType);

        ...
    }
```


SubscriberMethodFinder.java

```
// key: subscriber, value: subscriber所有的注册回调方法
private static final Map<Class<?>, List<SubscriberMethod>> METHOD_CACHE = new ConcurrentHashMap<>();

// 获取订阅者类里所有的注册回调方法
List<SubscriberMethod> findSubscriberMethods(Class<?> subscriberClass) {
        List<SubscriberMethod> subscriberMethods = METHOD_CACHE.get(subscriberClass);
        if (subscriberMethods != null) {
            return subscriberMethods;
        }

        if (ignoreGeneratedIndex) {
            subscriberMethods = findUsingReflection(subscriberClass);
        } else {
        	// 获取回调方法具体过程
            subscriberMethods = findUsingInfo(subscriberClass);
        }
        if (subscriberMethods.isEmpty()) {
            throw new EventBusException("Subscriber " + subscriberClass
                    + " and its super classes have no public methods with the @Subscribe annotation");
        } else {
            METHOD_CACHE.put(subscriberClass, subscriberMethods);
            return subscriberMethods;
        }
    }    


// 反射获取 Event 注解信息
private void findUsingReflectionInSingleClass(FindState findState) {
        Method[] methods;
        try {
            // This is faster than getMethods, especially when subscribers are fat classes like Activities
            methods = findState.clazz.getDeclaredMethods();
        } catch (Throwable th) {
            methods = findState.clazz.getMethods();
            findState.skipSuperClasses = true;
        }
        
        for (Method method : methods) {
            int modifiers = method.getModifiers();
            
        	// 回调方法必须是public
            if ((modifiers & Modifier.PUBLIC) != 0 && (modifiers & MODIFIERS_IGNORE) == 0) {
                Class<?>[] parameterTypes = method.getParameterTypes();
                
                // 回调方法的参数必须只有1个
                if (parameterTypes.length == 1) {
                
                    Subscribe subscribeAnnotation = method.getAnnotation(Subscribe.class);
                    if (subscribeAnnotation != null) {
                        Class<?> eventType = parameterTypes[0];
                        if (findState.checkAdd(method, eventType)) {
                            ThreadMode threadMode = subscribeAnnotation.threadMode();
                            
                            //将回调方法相关信息组装成 SubscriberMethod
                            findState.subscriberMethods.add(new SubscriberMethod(method, eventType, threadMode, subscribeAnnotation.priority(), subscribeAnnotation.sticky()));
                        }
                    }
                } else if (...) {
                    ...
                }
            } else if (...)) {
                ...
            }
        }
    }
    
```

PS：Map<Class<?>, CopyOnWriteArrayList<Subscription>> subscriptionsByEventType
key是Event类型(示例中的StatusEvent和DescribeEvent)，value是使用了该Event的类，为List类型。说明，一个Event可以对应多个subscriber。

### 2. post过程：

1）根据post参数中的Event类型，找到注册了该Event的类（即Subscriber）及onEvent系列回调方法；

2）使用Method.invoke()，调用Subscriber的回调方法；


EventBus.java

```
private boolean postSingleEventForEventType(Object event, PostingThreadState postingState, Class<?> eventClass) {

        CopyOnWriteArrayList<Subscription> subscriptions;
        synchronized (this) {
        	// 根据Event，获取subscriber
            subscriptions = subscriptionsByEventType.get(eventClass);
        }
        if (subscriptions != null && !subscriptions.isEmpty()) {
        
        	// 遍历所有subscriber
            for (Subscription subscription : subscriptions) {
                postingState.event = event;
                postingState.subscription = subscription;
                boolean aborted = false;
                try {
                    postToSubscription(subscription, event, postingState.isMainThread);
                    aborted = postingState.canceled;
                } finally {
                    postingState.event = null;
                    postingState.subscription = null;
                    postingState.canceled = false;
                }
                if (aborted) {
                    break;
                }
            }
            return true;
        }
        return false;
    }

// 根据ThreadMode，在对应线程执行回调
 private void postToSubscription(Subscription subscription, Object event, boolean isMainThread) {
        switch (subscription.subscriberMethod.threadMode) {
            case POSTING:
                invokeSubscriber(subscription, event);
                break;
            case MAIN:
                if (isMainThread) {
                    invokeSubscriber(subscription, event);
                } else {
                    mainThreadPoster.enqueue(subscription, event);
                }
                break;
            case BACKGROUND:
                if (isMainThread) {
                    backgroundPoster.enqueue(subscription, event);
                } else {
                    invokeSubscriber(subscription, event);
                }
                break;
            case ASYNC:
                asyncPoster.enqueue(subscription, event);
                break;
            default:
                throw new IllegalStateException("Unknown thread mode: " + subscription.subscriberMethod.threadMode);
        }
    }

```

四种ThreadMode:

* POSTING：发送和接收在同一线程
* MAIN：无法发送在哪个线程，接收都在UI线程；言外之意，发送要是在UI线程，接收和发送在同一线程
* BACKGROUND：接收在异步线程；如果发送在异步线程，接收和发送在同一线程
* ASYNC：接收在异步线程，且和发送不在同一线程，即使发送也在异步线程


### 3. 注解Subscribe.priority作用

因为多个subscriber可以注册同一Event，比如Demo中EventBusActivity和PostEventActivity两个subscriber都注册了StatusEvent。

priority的作用就是决定多个subscriber中的回调方法的执行顺序。

demo中，PostEventActivity注册StatusEvent指定了priority = 1，而EventBusActivity没指定StatusEvent的priority（默认0），所以post(StatusEvent)时，PostEventActivity先回调。

这个实现原理，是register()过程中，就已经按照priority将subscriber按顺序放到subscriptionsByEventType中。所以取subscriber进行回调时，自然就按照priority顺序回调了。


### 4. ConcurrentHashMap

核心数据结构subscriptionsByEventType，就是一个ConcurrentHashMap。

具体内容见：[link](../../DataStructrue/ConcurrentHashMap.md)

