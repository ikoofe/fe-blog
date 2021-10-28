---
title: 如何判断 Canvas 画布中的多边形是否被点击
date: 2021-10-26 15:18:11
tags: canvas
---


在 Canvas 实现的应用中，会有这样的一类场景，当点击画布中的某个几何图形时，要触发一些交互操作。Canvas 不支持为画布内的图形元素添加事件，这需要借助数学知识来解决这个问题。

## 获取坐标 

虽然，Canvas 不支持为画布内的图形元素添加事件，但是我们可以监听 Canvas 元素自身的点击事件，来计算出点击位置在画布上的坐标。

```JavaScript
canvas.addEventListener('click', (event) => {
  const rect = canvas.getBoundingClientRect();
  const x = event.clientX - rect.left;
  const y = event.clientY - rect.top;
  

  const point = {
    x,
    y,
  }

  console.log(point); // 打印点击位置坐标
})
```

如果用户想点击某个图像，那么他的点击位置一定会落在图形的内部。为了简化问题，这里将只讨论多边形的场景。判断一个点是否在多边形内，有交叉数 (Crossing Number) 和环绕数 (Winding Number) 两种方法。

## 交叉数法

交叉数法：以某一点做射线，如果该射线与多边形的边相交的次数为奇数时，则该点在多边形内部，否则在多边形外部。如下图所示：

![](https://img-blog.csdnimg.cn/20200620185329406.gif)


```JavaScript
function getCrossingNumber(point, lines) {
  let count = 0;
  for (let i = 0; i < lines.length; i += 1) {

    // o, d 是多边形某条边的起点和终点
    const { o, d } = lines[i]; 

    // 起点和终点位于水平射线的两侧才会有交点
    if ((o.y > point.y) ^ (d.y > point.y)) {

      // x = (y - y0) / k + x0
      const x = (point.y - o.y) * (d.x - o.x) / (d.y - o.y) + o.x;

      if (x > point.x) {
        count += 1;
      }
    }
  }

  return count;
}

```

Fabric.js 这个库就是使用交叉数法来判断一个点是否在多边形内，具体可以查看代码 https://github.com/fabricjs/fabric.js/blob/8ecbdb10f797ce07fb4dccca348fe63ff1558b62/dist/fabric.js#L16499

交叉数法在某些场景下，得出的结果会出现错误，相对来说，环绕数法会更准确一些。


## 环绕数法

环绕数法：以某一点做水平向右的射线，如果多边形的某条边的从下往上穿过该射线，则环绕数加一；如果多边形的某条边的从上往下穿过该射线，则环绕数减一；最终的环绕数如果不为 0 则该点在多边形内部，否则在多边形的外部。如下图所示：

![](https://img-blog.csdnimg.cn/20200630200316396.gif)

对于人来说，判断从上往下（或从下往上）穿过射线相对来说比较容易，在代码实现上会有一些复杂，因此需要将这种上下关系转换为左右关系。如下图所示，如果一条边向上穿过射线，那么 P 点在边 AB 的左侧；而对于一条向下的边，P 点在边 AB 的右侧。

![](https://i.loli.net/2021/10/27/gE3qhDIZelcGLVH.png)

使用右手定则，当 P 点在 AB 的左侧时， AB 和 AP 的法向量向外；当 P 点在 AB 的右侧时， AB 和 AP 的法向量向里。AB 和 AP 的法向量可以通过向量的外积（叉乘）来计算：

```
向量 AB = (x1, y1, 0)
向量 AP = (x2, y2, 0)

AB × AP = (x1y2 - x2y1)k

其中，k 是 z 轴单位向量，x1y2 - x2y1 的正负代表方向
```
进而得到以下结果：

- 当 x1y2 - x2y1 > 0 时，P 在 AB 左侧，环绕数 +1
- 当 x1y2 - x2y1 = 0 时，P 在 AB 上，环绕数不变
- 当 x1y2 - x2y1 < 0 时，P 在 AB 右侧，环绕数 -1

## 总结

通过上面的两个算法，都可以判断出 Canvas 画布上点击的位置是否在某个多边形上，然后触发相关交互即可。其中，环绕数法理解起来需要一些数学知识，但是代码实现并不复杂。