# Bitmap加载优化

一张图片从assert目录下加载到内存和从drawable目录下加载到内存有什么区别？

在assert目录下放一张图片，```image_asserts.png```

然后尝试使用BitmapFactory解码这个图片得到一个Bitmap，通过Bitmap中byteCount方法来获取它所占的字节数，这样我们就可以比较这些Bitmap所占用的内存大小了。

```
val bitmapAssert = BitmapFactory.decodeStream(assets.open("image_assets.png"))
Log.d("gaohui","bitmapAssert bitmap size: " + bitmapAssert.byteCount)
```

输出结果是：
```
gaohui: bitmapAssert bitmap size: 186624
```

接着我们来看下把同一张图片放在drawable-xxh目录下会是什么效果？
```
val bitmapXxh = BitmapFactory.decodeResource(resources, R.drawable.image_xxh, options)

Log.d("gaohui","bitmapXxh size: " + bitmapXxh.byteCount)
```

运行后，打印结果是：
```
gaohui: bitmapXxh size: 156816
```
从上面的打印结果看发现相比于Assert目录下加载的Bitmap，在drawable-xxxh目录下解码出来的Bitmap字节数要小。

我们再把这张图片放到drawable-xxxh目录下，看看是不是还会有改变？

```
val bitmapXxxh = BitmapFactory.decodeResource(resources, R.drawable.image_xxxh, options)

Log.d("gaohui","bitmapXxxh size: " + bitmapXxxh.byteCount)
```

打印结果：
```
gaohui: bitmapXxxh size: 156816
```

打印后发现竟然和drawable-xxh目录下的一样。

这里就感觉比较奇怪，按照Android官网的说法，不同drawable目录下的加载出来的Bitmap应该是不一样的。

这里我们尝试修改成下面这样：
```
//val bitmapXxxh = BitmapFactory.decodeResource(resources, R.drawable.image_xxxh, options)
val bitmapXxxh = BitmapFactory.decodeResource(resources, R.drawable.image_xxxh, BitmapFactory.Options())
Log.d("gaohui","bitmapXxxh size: " + bitmapXxxh.byteCount)
```
这样处理的打印结果就是正确的了。
```
gaohui: bitmapXxxh size: 88804
```
上面drawable-xxxh目录下和drawable-xxh目录下同一张图片decode出来的Bitmap占用内存一样的原因主要因为BitmapFactory.Options()复用导致的。
后面的修改重新new了一个BitmapFactory.Options()，发现Bitmap的字节数变小了，实际上这才是正确的占用内存大小。

那么接下来我们要解决的问题是为什么不同目录下加载的同一张图片但是Bitmap占用的大小却不一样呢？

关于Bitmap的内存占用这里有个计算公式：

> 图片内存大小 = (图片width * scale + 0.5) * (图片height * scale + 0.5)  * 每个像素字节

而这个scale是这样计算的：
> scale = 设备的屏幕密度 / drawable目录设定的屏幕密度

设备的屏幕密度跟手机有关，不同的手机可能表现不一样。

使用这个就可以获得设备的屏幕密度。
```
 getResources().getDisplayMetrics().densityDpi
```

不同的drawable目录下屏幕密度：

目录 | 屏幕密度
----|---
drawable-ldpi | 120dpi
drawable-mdpi | 160dpi
drawable-hdpi | 240dpi
drawable-xhdpi | 320dpi
drawable-xxhdpi | 480dpi
drawable-xxxhdpi | 640dpi

当我们使用decodeResource方法读取drawable目录下面的图片时，会根据手机的屏幕密度，到对应的文件夹中查找该图片。若该图片存在于其他目录下，系统先对该图片进行缩放处理，再显示。

最后我们来看下每个像素占用的字节。

BitmapFactory.Options中的inPreferredConfig表示的是图片解码格式，即图片的每个像素占用的字节数。

常见的解码格式有以下几种：
* Bitmap.Config.ARGB_8888: ARGB 4个通道，每个像素4个字节。部分图片显示时不需要Alpha，所以可使用565格式。
* Bitmap.Config.ARGB_4444: 官方标注Deprecated。ARGB 4个通道，每个像素2个字节。
* Bitmap.Config.RGB_565：RGB 3个通道，每个像素2个字节。
* Bitmap.Config.ALPHA_8：每个像素1个字节。主要用于Alpha通道模板，相当做一个染色。图像渲染两次，虽然节省内存，但增加了绘制开销


介绍了这么多我们来自己尝试计算下图片的内存大小是否和打印出来的结果一样。

```
    val scaleXxh = densityDpi/480f
    Log.d("gaohui","scaleXxh: $scaleXxh")
    //图片高度和宽度都是216
    val byteCountXxh = (scaleXxh * 216 + 0.5)* (scaleXxh * 216 + 0.5) * 4
    Log.d("gaohui", "Calculator bitmapXxh size: $byteCountXxh")

     val scaleXxxh = densityDpi/640f
    //图片高度和宽度都是216
    val byteCountXxxh = (scaleXxxh * 216  + 0.5f)* (scaleXxxh * 216 + 0.5) * 4
    Log.d("gaohui", "Calculator bitmapXxXh size: $byteCountXxxh")

```

打印出来结果如下所示：
```
gaohui: Calculator bitmapXxh size: 157609.0
gaohui: Calculator bitmapXxXh size: 88804.0
```
这里有人又会问问题了，你这个不对，drawable-xxh下通过getByteCount拿到的大小是156816，而你这里是157619，明显不对啊。

其实这里主要是精度问题导致的差距。

具体Bitmap的decode流程可以看下这篇文章：[Android中Bitmap占用内存计算](https://www.jianshu.com/p/578357ab6838)

这里我说下引起上述精度问题的差距主要在BitmapFactory.cpp中doDecode方法中，BitmapFactory.cpp中的doDecode方法是BitmapFactory.java上层调用过来的。

在BitmapFactory.cpp中的doDecode方法中：
```
static jobject doDecode(JNIEnv* env, SkStream* stream, jobject padding,
        jobject options, bool allowPurgeable, bool forcePurgeable = false,
        bool applyScale = false, float scale = 1.0f) {
        
        ...
        int scaledWidth = decoded->width();
        int scaledHeight = decoded->height();
        if (willScale && mode != SkImageDecoder::kDecodeBounds_Mode) {
        scaledWidth = int(scaledWidth * scale + 0.5f);
        scaledHeight = int(scaledHeight * scale + 0.5f);

        //...
    }
    }
    
```
这里计算Bitmap宽度的地方会先加个0.5f然后再转成int，而这里的scaledWidth和scaledHeight会被用到内存计算那里使用。

我们来改下这个部分的代码：

```
 val scaleXxh = densityDpi/480f
//图片高度和宽度都是216
val w =  (scaleXxh * 216 + 0.5).toInt()
val h = (scaleXxh * 216 + 0.5).toInt()
val byteCountXxh = w * h * 4
Log.d("gaohui", "Calculator bitmapXxh size: $byteCountXxh")
```

输出结果，这样的话就跟最开始通过Bitmap的getByteCount方法获取是一致的了。
```
gaohui: Calculator bitmapXxh size: 156816
```

Options.inSampleSize 图片解码尺寸

使用inSampleSize可以使解码后的图片尺寸（宽高）缩小，从而节省内存。
比如，inSampleSize=4，解码后图片宽高分别是原图的1/4。
原理是，解码器会根据这个值，每几个像素读入一次。

3.Options.inBitmap 重用内存

Android3.0引入了此字段。使用了此字段，decode方法会尝试重用一个已经存在的位图。这就意味着位图内存被重用了，从而改善性能，并且没有内存分配和释放的过程。
