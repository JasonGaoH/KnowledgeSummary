# UI
* SurfaceView VS TextureView
* ConstraintLayout

##### SurfaceView VS TextureView
相同点：都继承于View，可在独立线程绘制和渲染。

不同点：

* SurfaceView：嵌入视图层级内的绘制界面，是一种独立的View，更像是Window，不能做缩放、平移、画圆角等一般View形式操作；
* TextureView：更像是一般的View，可以做缩放、平移等View形式操作；

