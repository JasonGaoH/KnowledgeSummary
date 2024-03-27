

Alpha 是图形界面开发中常用的特效，通常我们会使用以下代码来实现 Alpha 特效:
```java
view.setAlpha(0.5f);
View.ALPHA.set(view, 0.5f); 
ObjectAnimator.ofFloat(view, "alpha", 0.5f).start(); 
view.animate().alpha(0.5f).start(); 
view.setAnimation(new AlphaAnimation(1.0f, 0.5f));
```

其效果都等同于:

canvas.saveLayer(l, r, t, b, 127, Canvas.CLIP_TO_LAYER_SAVE_FLAG);

所以常见的 alpha 特效是通过将图像绘制到 offscreen buffer 中然后显示出来，这样的操作是非常消耗资源 的，甚至可能导致性能问题，在开发过程中我们可以通过其他方式避免创建 offsreen buffer。

## TextView

对于 TextView 我们通常需要文字透明效果，而不是 View 本身透明，所以，直接设置带有 alpha 值的 TextColor 是比较高效的方式。

```java
// Not this 
textView.setAlpha(alpha);
// 以下方式可以避免创建 offscreen buffer
int newTextColor = (int) (0xFF * alpha) << 24 | baseTextColor & 0xFFFFFF; textView.setTextColor(newTextColor);

```

## ImageView
同样的对于只具有 src image 的 ImageView，直接调用 setImageAlpha()方法更为合理。

```java
// Not this, setAlpha 方法由 View 继承而来，性能不佳 
imageView.setAlpha(0.5f);
// 使用以下方式时，ImageView 会在绘制图片时单独为图片指定 Alpha // 可以避免创建 offScreenBuffer
imageView.setImageAlpha((int) alpha * 255);
```

## CustomView

类似的，自定义控件时，应该直接去设置 paint 的 alpha。

```java

// Not this 
customView.setAlpha(alpha);
// But this
paint.setAlpha((int) alpha * 255); canvas.draw*(..., paint);
```

同时 Android 提供了 hasOverlappingRendering()接口，通过重写该接口可以告知系统当前 View 是否存在内容重叠的情况，帮助系统优化绘制流程，原理是这样的:对于有重叠内容的 View，系统简单粗暴的使用 offscreen buffer 来协助处理。当告知系统该 View 无重叠内容时，系统会分别使用合适的 alpha 值绘制每一层。

```java

/**
* Returns whether this View has content which overlaps. This function, intended to be
* overridden by specific View types, is an optimization when alpha is set on a view. If
* rendering overlaps in a view with alpha < 1, that view is drawn to an offscreen buffer
* and then composited it into place, which can be expensive. If the view has no overlapping * rendering, the view can draw each primitive with the appropriate alpha value directly.
* An example of overlapping rendering is a TextView with a background image, such as a
* Button. An example of non-overlapping rendering is a TextView with no background, or
* an ImageView with only the foreground image. The default implementation returns true;
* subclasses should override if they have cases which can be optimized. *
* @return true if the content in this view might overlap, false otherwise. */
public boolean hasOverlappingRendering() { 
    return true;
}
```

最后引用 Chet Haase 的一句话作为总结
“You know what your view is doing, so do the right thing for your situation.”


> 参考

[给 App 提速：Android 性能优化总结](https://juejin.im/entry/5aa24187518825557207f8e2)