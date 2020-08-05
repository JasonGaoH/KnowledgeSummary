之前一直负责小红书的关注Feed的迭代工作，因为一直是在完成新功能的迭代工作，对于Feed的性能和消费体验就没有特别关注，加上对于这块业务的一些监控也没有落地，所以长期对于这块的性能基本上就是一个忽视的状态。随着业务越来越复杂，功能越来越多，收到好多反馈都是说关注页面的滑动体验很不好，于是决定对关注Feed做一个性能优化。

这里先从优化说起，之前在网上看到一个关于优化的分类，觉得很有道理。

请看下面这张图：
![](https://raw.githubusercontent.com/JasonGaoH/Images/master/uPic/optimization_category.png)

### 关于优化
优化整体可以分为两大类，软优化和硬优化。那么什么是软优化呢？软优化并不会改变某段代码的执行效率，而是通过调整这段代码的执行时机，来优化整体代码的运行性能。软优化里由分为三大类，分别为预加载，异步加载和懒加载。

#### 软优化
- 预加载
这几种加载方式都很好理解，改变的都是执行时机，预加载就是提前加载，把一些重要的数据提前加载好，等到合适的时机，就可以直接展示给用户，而不是在这个合适的时机才去加载数据，这种方式给用户地感觉是加载变快了。
- 异步加载
而异步加载在Android中则很常见了，移动应用中有主线程的概念，主线程需要实时地对用户的操作予以反馈，这种反馈要非常及时，否则就会让用户觉得卡顿。因此，主线程的工作不宜有耗时操作，而且对于一些执行比较慢的代码，而且不需要同步执行的代码，我们可以考虑将它放到异步线程去执行。
- 懒加载
最后就是懒加载了，懒加载就是把一些不是需要立即展示的东西往后放，因为我们的手机性能，内存是有限的，有些功能用户不是需要马上看到，则可以考虑在后续的某个适当的时机执行加载。

#### 硬优化
接下来说说硬优化，硬优化这个东西就比较偏细节了。一段逻辑，我们用两层循环实现，功能肯定是OK的，但是如果在实现功能的情况下，我们用一层循环就可以解决了，这样不就提高了这段代码的运行效率了。这种方式的优化就是硬优化了，关于硬优化往往没有具体的方法，很多都是各种见仁见智，通过调整代码的逻辑来提高运行效率都可以称为硬优化。

#### Feed流上的问题发现和解决
很多人会说，上面说了这么多软优化和硬优化，有什么用呢？别急，我们慢慢来说。上面说的算是一种对于优化的指导思想吧，下次我们遇到类似的问题可以从这几个维度来进行分析。

我们要做优化，首先肯定是要发现问题，也就是对症下药。

我之前主要处理的是Feed流的卡顿优化，所以我基本上都是通过Android Studio的Profiler工具来查看问题，现在Profiler的体验相对之前功能算是更加强大了，我每次都是使用Profiler来尝试抓一下Feed流滑动时的代码执行情况。
然后一个一个查看滑动过程中我们的耗时是在哪些地方。

##### 通过Profiler发现的问题
通过Profiler发现了以下几点问题：
1. 发现有一些比较耗时的方法，比如图文笔记上的滤镜的回收方法比较耗时，针对这样的可以考虑做这个release操作放到异步里面去；
2. 另外也发现一些业务的打点会在OnBindViewHolder中调用的很频繁，虽然点位数据的发送是在异步线程里，但是在发送数据之前会有数据的拼装操作，多少会有些性能损耗，而且打点这一类数据完全可以放到异步线程中去，因为这种打点并不需要很高的实时性；
3. 对于一些跨进程的操作，比如从Context中拿Service获取WiFi状态，这种在onBind中回调是非常不好的，所以针对这个我们只需要构造一个广播来监听WIFI状态变化即可，然后每次只需要拿那个记录好的网络状态变量即可
4. 还有关于视频笔记的处理，我们需要根据RecyclerView滑动过程中当前item中的视频start，stop和prepare做相应的调整，正常情况下，当RecyclerView处于滑动状态是不应该去prepare和start的，而且也不应该创建播放器实例和Texture View的；
5. 最后，通过Profiler我们还发现TextView的setText是很耗时的，当我们复杂多行文本的情况时，TextView去setText的时候需要进行复杂的测量，在测量完之后还需要绘制，所以针对这个场景可以考虑使用StaticLayout对Feed流中的文本作预渲染，关于StaticLayout的教程网上也有不少，后面有时间我也会抽空写下关于这块的实践。

上面都是通过Profiler来发现的一些问题。

##### View层级
另外还有一些算是基础的优化吧。因为View的层级随着Feed流的业务变得越来越深，我们需要进可能地减少View的层级，同时也尽量减少View的个数，可以考虑使用ConstraintLayout，但是ConstraintLayout这个东西要慎用，对于一些迭代比较快的业务场景，使用ConstraintLayout会增加我们的维护成本，而且会有一些比较坑的问题。

关于View层级的缩减基本没啥可讲的，尽量用merge和ViewStub，这个就是根据实际情况去消除一些不必要的层级。

##### RecyclerView缓存的利用 
最后，我们也尝试了从RecyclerView缓存的角度来解决这个问题，RecyclerView默认会有四级缓存的概念，正常情况下，当我们的RecyclerView的卡片撑满屏幕的时候，我们进入该页面的时候，我们的RecyclerView会先创建ViewHolder，然后再bind这个ViewHolder，然后当我们向下滑动的时候，会再次创建一个ViewHolder，接着再执行Bind操作，只有当我们的RecycledView Pool的池子满了之后才不会继续创建ViewHolder，往后的滑动RecyclerViwe就只会执行bind操作了，这个时候我们发现这个OnCreateViewHolder需要从xml里去解析view，并且创建这个ViewHolder，这个创建对于我们的滑动是会有性能损耗，其实还是因为我们的这个卡片太复杂了。

所以，我们可以考虑提前创建一些ViewHolder，放到ViewHolder的池子里去，这样我们在滑动的时候，就不需要先Create再Bind了。

提前创建ViewHolder，在主线程Idle的状态下提前创建ViewHolder放到RecycledViewPool中，这个pool的size默认是5。
   ```java
	recyclerView.setRecycledViewPool(RecyclerView.RecycledViewPool().apply {
		LightExecutor.postIdle(Runnable {
			repeat(5) {
				//RecycledViewPool 默认size为5个，RecyclerView内部会有判断，如果满了不会create
				//注意这里取得是typePool的倒数第一个和第二个位置，需要保证注册的时候视频卡片和笔记在最后两个位置
				putRecycledView(mAdapter.createViewHolder(followFeedRecyclerView, mAdapter.typePool.size -1))
				putRecycledView(mAdapter.createViewHolder(followFeedRecyclerView, mAdapter.typePool.size -2))
			}
		})
	})
```

#### 结果
在做完这些优化之后，在Feed流上的卡顿就有很大改善了，另外，对于业务侧的指标也有一定的数据提升。

#### 后续
对于这块业务的性能监控可以实施起来，这样有利于我们尽早发现卡顿问题，另外，对于一些优化的点其实还可以继续深挖，比如前创建ViewHolder是否可以使用AsyncLayoutInflator进行inflate，还有针对文本的预渲染可以考虑抽成组件形式，方便其他业务使用。

感谢您的阅读，如果觉得我的文章对你有帮助，请帮忙点赞，有问题可以评论留言。
