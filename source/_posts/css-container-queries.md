---
title: CSS 容器查询（CSS Container Queries）
date: 2021-06-03 15:18:11
tags: 容器查询
---

在响应式布局布局中，经常使用媒体查询（Media Queries）检测视窗的宽高，实现自元素样式的自动调整。但是在一些页面设计中，元素的容器尺寸发生变化时，元素的样式也需要随之变化。很显然，媒体查询无法支持这种场景。为了解决这类问题，CSS 增加了一个新的特性 **容器查询（Container Queries）**。

**浏览配置**

目前，容器查询还没有被 Chrome 浏览器正式支持，需要手动开启容器查询功能。在 Chrome 浏览器中打开

`chrome://flags/#enable-container-queries`，然后找到对应配置后，将 `Default` 改为 `Enabled`。

![](https://s2.loli.net/2022/03/28/Cu6A3GfqWpg8VFs.png)

经过测试发现：
- Chrome 正式版「版本 100.0.4896.60（正式版本） (x86_64) 」设置后并未生效
- Chrome 开发版「版本 102.0.4972.0（正式版本）dev (x86_64)」设置后可以生效
- Chrome Canary「版本 102.0.4982.0（正式版本）canary (x86_64)」设置后可以生效

**容器查询**
为了支持容器查询，CSS 的新规范中增加了 `container-type`、`container-name` 和 `container` 三个属性。`container` 是 `container-type` 和 `container-name` 的简写属性，用来显式声明某个元素是一个查询容器，并且定义查询容器的类型（由 `container-type` 指定）和查询容器的名称（由 `container-name` 指定，使用反斜杠（`/`）隔开。

除此之外，要用 `@container` 来声明容器查询的样式，比如在下面的例子中，当 `<main>` 元素的宽度小于 800px 时，`<span>` 元素中的字体颜色被置为 `gray`：

```CSS
main {
  container: inline-size / name;
  /* 
  * 等价于
  * container-type: inline-size;
  * container-name: name;
  */
}

@container name (max-width: 800px) {
  span {
    color: gray;
  }
}
```

```HTML
<main>
  <span>koofe</span>
</main>
```

声明了 `container-type` 这个属性，就意味着告诉浏览器，在该元素上创建一个容器上下文，之后可能要查询此容器。
`container-type` 支持如下属性：

- `size` ：在容器的块轴（Block Axis）和内联轴（Inline Axis）上建立一个查询容器，用于尺寸查询，即查询容器块轴和内联轴的尺寸（block-size和inline-size）。对元素应用layout、style 和 size 限制
- `inline-size` ：在容器的内联轴（Inline Axis）上建立一个查询容器，用于内联轴尺寸（inline-size）查询。对元素应用layout、style 和 inline-size 限制
- `block-size` ：在容器的块轴（Block Axis）上建立一个查询容器，用于块轴尺寸（block-size）查询。对元素应用布局layout、style 和 block-size 限制
- `style` ：建立一个查询容器，用于样式查询
- `state` ：建立一个查询容器，用于状态查询

`container-name` 用来指定当前容器所对应的容器查询的名称，可以省略不设置该属性。`@container` 方法通过对被查询的元素应用大小和布局的限制来实现。任何具有尺寸和布局限制的元素都可以通过一个新的 `@container` 规则进行查询，其语法与现有的媒体查询类似。

**示例演示**

<iframe height="300" style="width: 100%;" scrolling="no" title="CSS 容器查询" src="https://codepen.io/tr1a/embed/qBppZZe?default-tab=html%2Cresult" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href="https://codepen.io/tr1a/pen/qBppZZe">
  CSS 容器查询</a> by tr1a (<a href="https://codepen.io/tr1a">@tr1a</a>)
  on <a href="https://codepen.io">CodePen</a>.
</iframe>

具体代码如下：

```HTML
  <div class="content">
    <div class="card">
      <div class="qrcode">
        <img src="https://s2.loli.net/2022/04/04/4VuT1aKLEZ3O6Mj.jpg" />
      </div>
      <div class="box">
        <div class="title">KooFE</div>
        <div class="about">介绍前端知识技巧，了解前端最新动态，欢迎大家关注！</div>
      </div>
    </div>
  </div>
```

```CSS
.content {
  container-type: inline-size;
  resize: horizontal;
  overflow: hidden;
  background: #efefef;
}

.card {
  display: flex;
  width: 600px;
  margin: 8px;
  padding: 8px;
  border-radius: 4px;
  background: #ffffff;
}

.box {
  padding-left: 16px;
}

.title {
  line-height: 36px;
  font-weight: 800;
  font-size: 20px;
}

.content .about {
  padding-top: 8px;
  line-height: 24px;
  color: #666666;
}

@container (max-width: 800px) {
  .card {
    width: 200px;
    flex-direction: column;
  }
  .box {
    padding: 0;
  }
  .about {
    display: none;
  }
}
```

目前，容器查询的标准处于**工作草案**（Working Draft）阶段，绝大部分浏览器并未支持该特性，所以也无法在实践开发中进行应用。容器查询会更好的支持页面的响应式设计，并且在一定程度上能够丰富组件的样式，让组件变得更加通用。