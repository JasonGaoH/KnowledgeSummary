##### SurfaceView VS TextureView
相同点：都继承于View，可在独立线程绘制和渲染。

不同点：

* SurfaceView：嵌入视图层级内的绘制界面，是一种独立的View，更像是Window，不能做缩放、平移、画圆角等一般View形式操作；
* TextureView：更像是一般的View，可以做缩放、平移等View形式操作；

从性能和安全性角度出发，使用播放器优先选SurfaceView。
1.在android 7.0上系统surfaceview的性能比TextureView更有优势，支持对象的内容位置和包含的应用内容同步更新，平移、缩放不会产生黑边。 在7.0以下系统如果使用场景有动画效果，可以选择性使用TextureView
2.SurfaceView优点及缺点优点：可以在一个独立的线程中进行绘制，不会影响主线程，使用双缓冲机制，播放视频时画面更流畅
缺点：Surface不在View hierachy中，它的显示也不受View的属性控制，所以不能进行平移，缩放等变换，也不能放在其它ViewGroup中。SurfaceView 不能嵌套使用
TextureView优点及缺点
优点：支持移动、旋转、缩放等动画，支持截图
缺点：必须在硬件加速的窗口中使用，占用内存比SurfaceView高，在5.0以前在主线程渲染，5.0以后有单独的渲染线程

