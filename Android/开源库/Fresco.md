# Frsco



##### 相关问题：
1. Fresco 下载模块如何设计？<br>
默认使用HTTPURLConnection，也支持使用OkHttp。下载类是NetworkFetcher。

2. Fresco 图片缓存模块如何设计？<br>
三级缓存：Bitmap缓存(5.0以前放Ashmen) + 未解码图片缓存 + 硬盘缓存

3. 硬盘缓存如何设计？<br>

4. SimpleDraweeView layout_width/height 为何不支持 wrap_content?<br>

5. DraweeView多层结构？<br>



##### 模块结构
```
demo
	compile project(':drawee-backends:drawee-pipeline')

drawee-pipeline (包含在drawee-backends目录中，同级还有drawee-volly)
	compile project(':drawee')
    compile project(':fbcore')
    compile project(':imagepipeline')
    
drawee
	compile project(':fbcore')
	
imagepipeline
	compile project(':fbcore')
	
fbcore
	最底层module，提供file, log, datasource, memory等基础功能
    
```

```
SimpleDraweeView.setImageUri(Uri, Context) 流程：

-> AbstractDraweeControllerBuilder.build(): AbstractDraweeController
-> AbstractDraweeControllerBuilder. buildController(): AbstractDraweeController
-> PipelineDraweeControllerBuilder. obtainController(): PipelineDraweeController
-> AbstractDraweeControllerBuilder. obtainDataSourceSupplier(): Supplier<DataSource<IMAGE>>
-> AbstractDraweeControllerBuilder. getDataSourceSupplierForRequest(): Supplier<DataSource<IMAGE>>
-> PipelineDraweeControllerBuilder. getDataSourceForRequest(): DataSource<CloseableReference<CloseableImage>>
-> ImagePipeline. fetchDecodedImage(): DataSource<CloseableReference<CloseableImage>>
-> ImagePipeline. submitFetchRequest(): DataSource<CloseableReference<CloseableImage>>
-> CloseableProducerToDataSourceAdapter. create(): DataSource<CloseableReference<T>>
-> AbstractProducerToDataSourceAdapter. construtor()
-> Producer. produceResults()


SimpleDraweeView attach到window时，执行如下流程：
-> DraweeView. onAttachedwindow()
-> DraweeHolder. onAttach()
-> DraweeHolder. attachController()
-> AbstrackDraweeController. onAttach()
-> AbstrackDraweeController. submitRequest()
-> AbstractDataSource. subscribe()

```


##### 主要类

```
ImagePipline：负责从网络、本地文件、本地资源加载图片，并将其解码到内存中供系统使用。

ImageRequest, 

Supplier<DataSource<T>>

DraweeController:

SimpleDraweeControllerBuilder(I):
DraweeController的构造器，主要实现类是PipelineDraweeControllerBuilder。
继承结构：
PipelineDraweeControllerBuilder -> 
AbstractDraweeControllerBuilder -> 
SimpleDraweeControllerBuilder


**AbstractDataSource:**
封装了Producer，
具体实现类 AbstractProducerToDataSourceAdapter

AbstractDataSource
	->	VolleyDataSource
	->	AbstractProducerToDataSourceAdapter
		->	CloseableProducerToDataSourceAdapter


**Producer:**
封装了Fetcher。
使用生产者消费者模式(Producer-Consumer)，获取不同来源的图片资源(cache, file, url)。

继承关系：
-> Producer (I)
	-> LocalFetchProducer
		-> LocalAssetFetchProducer
		-> LocalFileFetchProducer
		-> LocalResourceFetchProducer
	-> NetworkFetchProducer
	-> LocalVideoThumbnailProducer
	
	
		
**Fetcher:**
网络请求实现类，只在NetworkFetchProducer中使用。

继承关系：
-> NetworkFetcher (I)
	-> BaseNetworkFetcher
		-> HttpUrlConnectionNetworkFetcher (fresco默认使用的)
		-> OkHttpNetworkFetcher


```

# 核心模块

###### ImagePipeline
Fresco 中设计有一个叫做 Image Pipeline 的模块。它负责从网络，从本地文件系统，本地资源加载图片。为了最大限度节省空间和CPU时间，它含有3级缓存设计（2级内存，1级磁盘）。

Image pipeline 负责完成加载图像，变成Android设备可呈现的形式所要做的每个事情。

大致流程如下:

* 检查内存缓存，如有，返回
* 后台线程开始后续工作
* 检查是否在未解码内存缓存中。如有，解码，变换，返回，然后缓存到内存缓存中。
* 检查是否在磁盘缓存中，如果有，变换，返回。缓存到未解码缓存和内存缓存中。
* 从网络或者本地加载。加载完成后，解码，变换，返回。存到各个缓存中。


既然本身就是一个图片加载组件，那么一图胜千言。
![](https://www.fresco-cn.org/static/imagepipeline.png)



###### Drawees
Fresco 中设计有一个叫做 Drawees 模块，它会在图片加载完成前显示占位图，加载成功后自动替换为目标图片。当图片不再显示在屏幕上时，它会及时地释放内存和空间占用。
<br>
<br>
<br>


# 特性
### 1. 内存

[三级缓存]

```
**1. Bitmap缓存**
Bitmap缓存存储Bitmap对象，这些Bitmap对象可以立刻用来显示或者用于后处理

在5.0以下系统，Bitmap缓存位于Ashmem，这样Bitmap对象的创建和释放将不会引发GC，更少的GC会使你的APP运行得更加流畅。在5.0以下，频繁GC将会显著地引发界面卡顿。

Ashmem存储区域：它是一个不在Java堆区的一片存储内存空间，它的管理由Linux内核驱动管理，不必深究，只要知道这块存储区域是别于堆内存之外的一块空间就行了，且这块空间是可以多进程共享的，GC的活动不会影响到它。

5.0及其以上系统，相比之下，内存管理有了很大改进，所以Bitmap缓存直接位于Java的heap上。

当应用在后台运行时，该内存会被清空。

**2. 未解码图片的内存缓存**
这个缓存存储的是原始压缩格式的图片。从该缓存取到的图片在使用之前，需要先进行解码。

如果有调整大小，旋转，或者WebP编码转换工作需要完成，这些工作会在解码之前进行。

**3. 磁盘缓存**
和未解码的内存缓存相似，磁盘缓存存储的是未解码的原始压缩格式的图片，在使用之前同样需要经过解码等处理。

和Bitmap缓存不一样，APP在后台时，内容是不会被清空的。即使关机也不会。用户可以随时用系统的设置菜单中进行清空缓存操作。
```


##### 使用三级缓存：Bitmap缓存 + 未解码图片缓存 + 硬盘缓存
其中前两个就是内存缓存，Bitmap缓存根据系统版本不同放在了不同内存区域中，而未解码图片的缓存只在堆内存中，Fresco分了两步做内存缓存，这样做有什么好处呢？好处是加快图片的加载速度。

Fresco的加载图片的流程为：
1）查找Bitmap缓存中是否存在，存在则直接返回Bitmap直接使用；
2）不存在则查找未解码图片的缓存，如果存在则进行Decode成Bitmap然后直接使用并加入Bitmap缓存中；
3）如果未解码图片缓存中查找不到，则进行硬盘缓存的检查，如有，则进行IO、转化、解码等一系列操作，最后成Bitmap供我们直接使用，并把未解码（Encode）的图片加入未解码图片缓存，把Bitmap加入Bitmap缓存中，如硬盘缓存中没有，则进行Network操作下载图片，然后加入到各个缓存中。

既然Fresco使用了三级缓存，而有两级是内存缓存，所以当我们的App在后台时或者在内存低的情况下在onLowMemory()方法中，我们应该手动清除应用的内存缓存，我们可以使用下面的方式：

```
ImagePipeline imagePipeline = Fresco.getImagePipeline();
//清空内存缓存（包括Bitmap缓存和未解码图片的缓存）
imagePipeline.clearMemoryCaches();
//清空硬盘缓存，一般在设置界面供用户手动清理
imagePipeline.clearDiskCaches();

//同时清理内存缓存和硬盘缓存
imagePipeline.clearCaches();
```


### 2. 硬盘缓存

### 3. MVC结构
```
DraweeView			--	View
DraweeController	-- Controller
DraweeHierarchy		-- Model
```

作用：
DraweeView用来显示顶层视图（getTopLevelDrawable()）。
DraweeController控制加载图片的配置、顶层显示哪个视图以及控制事件的分发。 
DraweeHierarchy意为视图的层次结构，用来存储和描述图片的信息，同时也封装了一些图片的显示和视图层级的方法。

###### DraweeHolder
DraweeHolder是协调DraweeView、DraweeHierarchy、DraweeController这三个类交互工作的核心类。


参考：
* [Github 地址](https://github.com/facebook/fresco)
* [中文官方文档](https://www.fresco-cn.org/)
* [优秀博客](http://blog.desmondyao.com/fresco-3-draweeview/#more)















