---
title: Canvas 用像素检测某点是否在图形内
date: 2021-10-26 15:18:11
tags: canvas
---


在上一篇文章中，我们提到了用 **交叉数法** 和 **环绕数** 来判断某个点是否在几何图形内。除了这两种方法外，本文将介绍另外一种基于像素的颜色值的检测方案，并通过一些源码来解答某同学在评论区提到的 Create.js。


## 基本原理

当用户点击 Canvas 画布时，通过点击位置的像素可判断是否点击到某个图形上。例如，在下图中矩形区域的颜色是橘黄色，而矩形区域外面的颜色是透明的；如果计算出某点 P 的颜色是橘黄色，我们就可以推断出 P 点是落在了矩形上。

![](https://i.loli.net/2021/10/28/MFz6eopbLXf2g98.png)

上面介绍的是理想情况，实际情况要比这复杂得多。比如下图中，当 Canvas 渲染的图形颜色都一样时，这就没有办法做判断了。

![](https://i.loli.net/2021/10/28/g9Dbs1iwfV2BUup.png)


在大多数的 Canvas 库中（比如 Fabric.js、Create.js）通常会将图形的属性、状态等数据存储于内存中，在下次渲染时直接取出来即可使用。同理，有了这些数据，我们也可以在一个新的 Canvas 中绘制某个图形，这样就把问题简化到了上面提到的理想情况，通过颜色即可判断点落在了哪里。由于新的 Canvas 的背景是完全透明的，只需要对颜色的透明度进行检测即可，如果被检测的点透明度为 0 则该点未命中图形。

![](https://i.loli.net/2021/10/28/JFjThwAlq4Smn5N.png)


## 具体实现

Create.js 就是使用的这种透明度对比的方法，来判断某个点是否在某个图形上。这里把 Create.js 的一些代码展示出来，以方便大家的理解。

```JavaScript
/**
 * Tests whether the display object intersects the specified point in local coordinates (ie. draws a pixel with alpha > 0 at
 * the specified position). This ignores the alpha, shadow, hitArea, mask, and compositeOperation of the display object.
 * 
 * Please note that shape-to-shape collision is not currently supported by EaselJS.
 * @method hitTest
 * @param {Number} x The x position to check in the display object's local coordinates.
 * @param {Number} y The y position to check in the display object's local coordinates.
 * @return {Boolean} A Boolean indicating whether a visible portion of the DisplayObject intersect the specified
 * local Point.
*/
p.hitTest = function(x, y) {
  // DisplayObject._hitTestContext 是一个新的 Context
  var ctx = DisplayObject._hitTestContext;

  // 做矩阵变换，将检测的点移到 (0, 0) 的位置
  ctx.setTransform(1, 0, 0, 1, -x, -y);
  
  
  // 在新的 ctx 上绘制出图形
  this.draw(ctx, !(this.bitmapCache && !(this.bitmapCache._cacheCanvas instanceof WebGLTexture) ));

  // 获取 (0, 0) 位置的透明度，如果透明度大于 1 则被检测的点 (x, y) 在图形上
  var hit = ctx.getImageData(0, 0, 1, 1).data[3] > 1;

  ctx.setTransform(1, 0, 0, 1, 0, 0);
  ctx.clearRect(0, 0, 2, 2);
  return hit;
};
```

但是值得注意的是，当图形的填充颜色完全透明时，这种检测方法就会失效，需要做一些特殊的处理。Create.js 的解决方案是给这个透明图形绑定一个大小一致的不透明的图形，用这个图形来做检测。上面的代码在 https://github.com/CreateJS/EaselJS/blob/5366838f84423aa5b57c804f1b9e11e5fe7f9371/src/easeljs/display/DisplayObject.js#L1083 ，感兴趣可以自行查看。
