之前在项目中通过监听 RecyclerView 的scrollEvents来判断是否需要去加载更多。

因为我们在RecyclerView滑动的时候来判断是否需要加载更多的，这个时候scrollEvents这边会一直会有回调，所以为了防止在弱网下会出现多次请求的情况，项目中特地加了一个isLoading的flag来防止这种情况的发生。

```
fun loadMore(remainCount: Int = 6,loadFinish: () -> Boolean):Observable<PagingState> {
    scrollEvents()
            .filter { loadFinish() }
            .map {
                var realRemainCount = 0
                if (layoutManager is StaggeredGridLayoutManager) {
                    (layoutManager as StaggeredGridLayoutManager).let {
                        val last = IntArray(it.spanCount)
                        it.findLastVisibleItemPositions(last)
                        realRemainCount = it.itemCount - last.last()
                    }
                }
                if (layoutManager is LinearLayoutManager) {
                    (layoutManager as LinearLayoutManager).let {
                        realRemainCount = it.itemCount - it.findLastVisibleItemPosition()
                    }
                }

                when {
                    realRemainCount == 1 -> {
                        PagingState.END
                    }
                    realRemainCount <= remainCount -> {
                        PagingState.PAGING
                    }
                    else -> PagingState.OTHERS
                }
            }
}

loadMore(){
    isLoading
}.subcribe({
    //这个时候是网络请求，在doOnSubscribe的时候会isLoading设置为true，在doOnTerminal的时候会再将isLoading设置为false。
    loadMoreAction()
},{})

```

但是之前在解其他bug的时候我在scrollEvents的subscribe之前切了个线程，类似下面这样。

```
loadMore(){
    isLoading
}.observeOn(AndroidSchedulers.mainThread())
.subcribe({
    loadMoreAction()
},{})

```

但是这之后有用户反馈feed里出现了重复的笔记，通过log发现是用户在当时触发了两次一模一样的加载更多的请求。

所以又回头梳理这个部分的逻辑，于是写了下面这个demo试了一下。

```java

    private var isLoading = false
    var executor = Executors.newSingleThreadExecutor()

    fun testObserveOn() {
        Observable.fromArray(1,2)
            .filter {
                Log.d("gaohui", "filter")
                isLoading.not()
            }
            .observeOn(Schedulers.from(executor))
            .subscribe({
                isLoading = true
                Log.d("gaohui", "subscribe")
                Observable.just(it)
                    .subscribeOn(Schedulers.io())
                    .observeOn(Schedulers.from(executor))
                    .subscribe({
                        Log.d("gaohui", "it $it")
                        isLoading = false
                    },{})
            },{})
    }
```

如上代码，打印结果是这样的：
```
2020-04-03 14:08:40.101 14083-14153/? D/gaohui: filter
2020-04-03 14:08:40.102 14083-14153/? D/gaohui: filter
2020-04-03 14:08:40.102 14083-14153/? D/gaohui: subscribe
2020-04-03 14:08:40.103 14083-14153/? D/gaohui: subscribe
2020-04-03 14:08:40.103 14083-14153/? D/gaohui: it 1
2020-04-03 14:08:40.105 14083-14153/? D/gaohui: it 2
```

当我们把切换线程的observeOn代码注释掉之后
```
    fun testObserveOn() {
        Observable.fromArray(1,2)
            .filter {
                Log.d("gaohui", "filter")
                isLoading.not()
            }
            //.observeOn(Schedulers.from(executor))
            .subscribe({
                isLoading = true
                Log.d("gaohui", "subscribe")
                Observable.just(it)
                    .subscribeOn(Schedulers.io())
                    .observeOn(Schedulers.from(executor))
                    .subscribe({
                        Log.d("gaohui", "it $it")
                        isLoading = false
                    },{})
            },{})
    }

```

这个时候打印结果是下面这样的。

```
2020-04-03 14:10:51.057 14258-14258/com.gaohui.android.code.collection D/gaohui: filter
2020-04-03 14:10:51.057 14258-14258/com.gaohui.android.code.collection D/gaohui: subscribe
2020-04-03 14:10:51.057 14258-14258/com.gaohui.android.code.collection D/gaohui: filter
2020-04-03 14:10:51.058 14258-14340/com.gaohui.android.code.collection D/gaohui: it 1
```

这里我发现把observeOn切到主线程还是切到当前线程，只要执行了切线程的操作，我们的isLoading这个flag的控制效果就失效了。

observeOn的逻辑基本都在ObservableObserveOn.java 这个类里面。

在ObservableObserveOn.java中ObserveOnObserver这个内部类中的OnNext中，如下所示：

```
    @Override
        public void onNext(T t) {
            if (done) {
                return;
            }

            if (sourceMode != QueueDisposable.ASYNC) {
                queue.offer(t);
            }
            schedule();
        }

```

这里会把这个 t 放到queue 这个队列中。这个t 是我们Observable上游发送过来的数据源，这个queue 是在ObserveOnObserver这个内存类中定义的一个队列。

```
    static final class ObserveOnObserver<T> extends BasicIntQueueDisposable<T>
    implements Observer<T>, Runnable {

        private static final long serialVersionUID = 6576896619930983584L;
        final Observer<? super T> downstream;
        final Scheduler.Worker worker;
        final boolean delayError;
        final int bufferSize;

        SimpleQueue<T> queue;
        ...
    }
```

这个queue是用来存放当前线程需要执行的一些任务的。

接着在onNext后面调用了schedule方法。这个schedule就是准备执行这个Runnable。

```
 void schedule() {
            if (getAndIncrement() == 0) {
                worker.schedule(this);
            }
        }

```

这个Runnable里的run方法是下面这样的。

```
   @Override
        public void run() {
            if (outputFused) {
                drainFused();
            } else {
                drainNormal();
            }
        }
```

我们这里暂且不关注outputFused这个flag是啥意思，这里无论是drainFused方法还是drainNormal方法，
基本都是轮询queue里的消息，如果queue里有消息需要处理,就取出来执行。这里的逻辑和主线程的MessageQueue很类似，都是轮询，取消息进行处理。

看到这里我突然有点明白我们上面那里调用了observeOn之后为啥我们的flag isLoading的控制效果就没有生效了，observeOn之后的操作相当于是非阻塞的，我们切了一个线程，observeOn中的subscribe的代码时不会马上执行的，这样代码又回头去取新的数据流了，这个时候isLoading这个flag是没有改变的，所以导致触发了两次网络请求。

这里的observeOn 和 Android 里的 post 效果是一样的，observeOn 这个表示把这个之后的操作包装成一个Runnable 放到observeOn指定的这个Schedulers中去。