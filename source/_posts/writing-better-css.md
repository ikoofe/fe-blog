
---
title: 使用新的 CSS 特性，编写好的 CSS 代码
date: 2022-02-25 15:18:11
tags: CSS
---

> 本文翻译自 Aleksandr Hovhannisyan 的 [Writing Better CSS](https://www.aleksandrhovhannisyan.com/blog/writing-better-css/)。

在早期的 Web 开发中，页面的布局和定位通常要用表格和各种 hack 技术来实现。与那时相比，CSS 已经得到了长足的发展。如今，开发人员可以很轻松的编写出适用于所有主流浏览器的 CSS 代码，在实现复杂布局时也不会像以前那样绞尽脑汁。这不仅使响应式布局变得更容易，还可以通过删除冗余的代码来发布体积更小的样式。在本文中，我们将使用现代技术来降低代码的复杂程度，并编写出更好的 CSS 代码。

## 使用 :is 减少重复代码

在 CSS 中经常使用[选择器列表](https://developer.mozilla.org/zh-CN/docs/Web/CSS/Selector_list) 为多个选择器使用相同的样式。比如在下面的例子中，相同的样式被应用到了三个选择器上：

```CSS
.nav-link:focus,
.nav-link:hover,
.nav-link[aria-current="page"] {}
```

在上面的每一项选择器中，`.nav-link` 都重复出现。如果单独看这个例子，似乎也能接受这种重写法。但是当选择器变得更长的时候，比如增加了其他的修饰符，在下面示例中加上了 `:not`:

```CSS
.button:not(.disabled):focus,
.button:not(.disabled):hover {}
```

在 Sass 中，使用 `&` 连体符，减少了重复的代码，提高了开发体验：

```CSS
.nav-link {
  &:focus,
  &:hover,
  &[aria-current="page"] {}
}
```

但是，在 Sass 最终编译出的 CSS 中，重复的代码依然会存在。如果我们能减少重复，只指定一次选择器，那就太好了。

所有主流浏览器现在都支持 [`:is` 伪类函数](https://developer.mozilla.org/en-US/docs/Web/CSS/:is)。它接受用逗号分隔的选择器列表来匹配，允许我们编写更少的 CSS 来完成与以前相同的任务。

使用 `:is` 来重新实现最上面的例子：

```CSS
.nav-link:is(:focus, :hover, [aria-current="page"]) {}
```

不难发现，多个选择器被合并到一个选择器上，这个选择器的功能是与上面第一个等价的。

在上面的第二个示例中使用了 `:not`，现在对其进行改写：

```CSS
.button:not(.disabled):is(:focus, :hover) {}
```

这样，就不必重复其中的任何类和伪类，只需使用 `:is` 将它们书写一次，就可以实现一个选择器列表。

在 Sass 中使用 `:is` 也很方便，根本不需要重新键入选择器公共部分，只需使用 `&` 连体符：

```CSS
.nav-link {
  /* base styling */

  &:is(:focus, :hover, [aria-current="page"]) {
    /* active styling */
  }
}
```

虽然，上面提到的例子中，都是将 `:is` 链接到某个选择器之后，实际上 `:is` 是可以单独使用的。例如，在下面的 CSS 中，`:is` 被用来定位某些父选择器的直接后代：

```CSS
:is(.parent1, .parent2, .parent3) > * {}
```

如果不使用 `:is`，就要写很多重复代码：

```CSS
.parent1 > *,
.parent2 > *,
.parent3 > * {}
```

在使用 `:is` 时，有两点要关注一下，分别是优先级和容错性。 

### 优先级

`:is` 的优先级是由它的选择器列表中优先级最高的选择器决定的。也就是说，列表中所有的选择器都具有相同的优先级。在上面的第一个示例中，所有选择器的优先级是相同的。

```CSS
.nav-link:is(:focus, :hover, [aria-current="page"]) {}
```

在下面的例子中，由于 ID 选择器的存在，整个 `:is` 的优先级都被提高了，后面优先级低的选择器就被覆盖了。
```HTML
<div class="class"></div>
```

```CSS
div:is(#id, .class) {
  background: red;
}

/* This will always be overridden by the selector above */
div:is(.class, .another-class) {
  background: blue;
}
```

在这个例子中，div 元素的会被渲染成红色，而不是蓝色。因为列表中的 ID 选择器将其他选择器的优先级也提高了。

### 容错性

`:is` 使用了选择器容错解析 ([forgiving selector parsing](https://developer.mozilla.org/en-US/docs/Web/CSS/:is#forgiving_selector_parsing))，当选择器列表中的某一项不能被识别时，并不会影响到整个列表中的其他选择器。举个例子：

```CSS
.element:is(:focus, :unrecognized-selector) {}
```

虽然存在不能识别的选择器，`:is` 依然会解析列表中的其他参数，并将样式作用于合法的选择器，比如这个示例中的 `:focus` 会生效。在下面的列表选择器中，`:focus` 的样式不会生效。

```CSS
.element :focus,
.element :unrecognized-selector {
}
```


## 使用 :where 设置全局默认样式

与 `:is` 的用法一样，`:where` 也是接收选择器列表作为它的参数。

```CSS
.nav-link:where(:focus, :hover, [aria-current="page"]) {}
```

`:where` 和 `:is` 的不同之处在于，`:is` 
中选择器的优先级与列表中最高的优先级保持一致；而 `:where` 中选择器的优先级被设置为最低，权重值为 0。选择器优先级的权重值，如下所示：

- 内联样式：1000
- ID 选择器：100
- 类选择器、属性选择器等：10
- 元素选择器、伪元素选择器等：1
- 通配符、相邻选择器等：0


换句话说，`:is` 提高了列表参数中每个选择器优先级；`:where` 降低了列表参数中每个选择器优先级。下面两段代码中选择器的优先级是相同的。

```CSS
// 10 + 0
.nav-link:where(:focus, :hover, [aria-current="page"]) {}
```

```CSS
// 10
.nav-link {}
```

无论它的参数是多么复杂的选择器，它的优先级都是 0；比如在下面的代码中，选择器的优先级也是 0:

```CSS
where(#id:not(.very.high.specificity).more.classes) {}
```

由于优先级低，`:where` 声明的样式很容易被覆盖了，因此 `:where` 特别适合全局的 CSS样式重置的场景。Elad Schechter 在他实现的[现代 CSS 样式重置](https://elad2412.github.io/the-new-css-reset/)中，使用 `:where` 来为一些元素设置默认的样式。下面是他的部分代码实现：

```CSS
:where(ul, ol) {
  list-style: none;
}

:where(img) {
  max-width: 100%;
  height: auto;
}

/* etc */
```

在这份样式表中，选择器的优先级比较低，它们的样式很容易被其他选择器覆盖。当我们需要重写样式时，不需要特意提高那些选择器的优先级。

在 CSS 的实践中，重置样式通常会放在最前面，也很少使用元素选择器来声明样式，例如使用 BEM 技术。于是 `:where` 可以在一定程度上保证，定义的样式永远不会在优先级上遇到任何冲突。

另外，Adam Argyle 在他的文章 [:is 和 :where](https://web.dev/css-is-and-where/) 中提到，在一些公共库中，`:where` 优先级比较低这个特点，是非常有用的。因为，当用户需要自定义样式时，可以很方便地将公共库中的样式覆盖掉。

## 使用逻辑属性设置 RTL

如果你的应用只支持单一的语言（比如 en-US），而且没有做国际化相关工作，那么在编写 CSS 时，你可能并不需要关注文字 LTR（left-to-right，左对齐）和 RTL（right-to-left，右对齐）的差异。因此，我们可以很放心地去使用 `margin-left`、`padding-right`、以及绝对定位等属性。

但如果你的应用是国际化的，要支持多个地区的语言，那情况就完全不同了。在阿拉伯语、希伯来语等这些 RTL 的语言环境中，文本是从右向左读的，而不是从左向右读的。根据已有经验，页面上的大多数视觉元素要进行左右颠倒，并沿着与文本相同的方向进行布局（尽管很少有例外）。

实现 RTL 的传统方法：首先要从 LTR 的角度去编写 CSS，然后使用 `dir` 属性确定 RTL 样式的范围。这里通常使用物理维度的属性和值，这些属性和值的基本方向是固定的（top、right、 bottom 和 left）。下面是一个示例：

```CSS
.element {
  /* LTR CSS */
  margin-left: 8px;
}
html[dir="rtl"] .element {
  /* RTL CSS */
  margin-left: unset;
  margin-right: 8px;
}
```

在 LTR 模式下，我们的元素会有一个左外边距。在 RTL 模式下，元素需要的是一个右外边距。所以，我们取消左外边距，与此同时将右边距设置了相同的值。在页面中根据语言情况来决定哪种样式生效（RTL 版本具有更高的优先级）。但你会发现，当你处理越来越多的样式时，这个过程也变得越来越枯燥。为了实现国际化，我们额外开发了大量的重复代码。

下面列举了一些我们经常使用的物理属性：
- margin-[top|right|bottom|left]
- padding-[top|right|bottom|left]
- border-[top|right|bottom|left]

让我们感到欣慰的是，CSS 已经开始支持逻辑属性了。与物理属性相比，逻辑属性更易于适应不同的书写模式。在逻辑属性中没有物理维度的概念，只用 start 和 end 等来描述文本方向。因此，可以使用更少的代码来实现上面的功能：

```CSS
.element {
  margin-inline-start: 8px;
}
```

逻辑属性，通常由三部分组成，属性名称(`margin`)、流动方向 ([`block` 或 `inline`](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Logical_Properties#block_vs._inline))、起止方向 (`start` 或 `end`)，以及相关的子属性(例如 `color`、`width` 等)。所以与 `border-left-color` 对应的 逻辑属性是 `border-inline-start-color`。在 LTR 模式下，`start` 等价于 `left`，在 RTL 模式下，等价于 `right`。在这两种模式下，UI 样式与期望的效果是一致的，但只需要编写一个样式规则即可适应这两个模式。


在最近所有的 CSS 新特性中，逻辑属性可以算做是我最喜欢的一个。即使你的应用程序暂不支持 RTL，你仍然可以使用逻辑属性，因为它们可以无缝地用于 LTR。带来额外的好处是，如果未来有需要的话，可以快速实现应用程序的国际化。使用逻辑属性没有任何坏处，所需要的只是转变视角。

当你需要重构的代码越多，逻辑属性带来的收益越大。就在去年，我使用逻辑属性对一个已有项目重构（因为我们的应用程序支持 RTL），结果我删除了近 700 行不必要的 CSS。


### 逻辑属性示例

我们可以使用很多的逻辑属性和相关的属性值。在下面的章节中，列出了一些最常见并广泛使用的 CSS 逻辑属性，以及它们与物理属性的对应关系。值得注意的是，并不是所有的物理属性都有与之对应的逻辑属性。两个示例包括变换平移和长方体阴影偏移。

**边距**

我们可以把 margins、 padding、borders 替换成对应的逻辑属性，这样就会自动完成对 RTL 和 垂直书写模式的支持。下面的表格中列出了对应的关系：

| 物理属性 | 逻辑属性 |
| ------------------|----------------- |
|`[margin|padding|border]-left` | `[margin|padding|border]-inline-start`|
|`[margin|padding|border]-right` | `[margin|padding|border]-inline-end` |
|`[margin|padding|border]-top` | `[margin|padding|border]-block-start` |
|`[margin|padding|border]-bottom`	| `[margin|padding|border]-block-end` |
|`border-bottom-width` | `border-block-end-width` |
|`border-left-color` | `border-inline-start-color` |


如果从单个边框属性来看，逻辑属性的命名确实比较冗长。但与维护两套样式（一套用于 LTR，另一套用于 RTL）相比，它肯定就不那么冗长。

**定位**

包含 absolute、relative 和 fixed 在内的定位也可以通过逻辑属性来完成：

|Physical property	| Logical property |
|-------------------|------------------|
|top	              |inset-block-start |
|bottom	            | inset-block-end  |
|left	              |inset-inline-start|
|right	            |inset-inline-end  |

**宽高**

如果需要支持垂直书写模式，则有以下逻辑属性：

| 物理属性	| 逻辑属性 |
|-------------------|------------------|
|`width`            |`inline-size`     |
|`height`           |`block-size`      |

因为宽度是对称（没有方向性），对 RTL 样式没有任何影响。在使用它们时，我们不需要做额外的工作。

**逻辑属性值**

除了一些属性之外，还有一些属性值也是具有逻辑性的：

|物理规则 | 逻辑规则 |
|--------|--------|
|`text-align: right;` |	`text-align: end;`|
|`justify-content: left;`	| `justify-content: start;`|
|`float: left;`	| `float: inline-start;`|
|`float: right;` |	`float: inline-end;`|

浏览器支持情况

![](https://s2.loli.net/2022/02/08/Hja6438lSnQuFsY.jpg)

## 使用 clamp 媒体查询

在下面的示例中，为了调整 element 的字体大小，给 element 设置了两段字体样式代码：

```CSS
.element {
  font-size: 1rem;
}
@media screen and (min-width: 768px) {
  .element {
    font-size: 1.25rem;
  }
}
```

如果不使用媒体查询，这些样式上的变化是无法实现的。对于间距、字体大小等数字属性值，实际上可能需要的是在两个点之间进行线性缩放，而不是让它从一个离散值变为另一个离散值。如果允许使用 CSS 的新特性，可以利用 [`clamp` 函数](https://developer.mozilla.org/en-US/docs/Web/CSS/clamp())来实现属性值在最小值和最大值之间变化。


### clamp 函数

`clamp` 是一个 CSS 函数，它接收三个用逗号分隔的表达式作为参数，按最小值、首选值、最大值的顺序排列：

```CSS
.element {
  property: clamp(<min>, <preferred>, <max>);
}
```

当首选值比最小值要小时，则使用最小值；当首选值介于最小值和最大值之间时，用首选值；当首选值比最大值要大时，则使用最大值。因此，首选值被限制在一个上限和下限之间。这相当于，将 `min` 和 `max` 函数关联到了一起，clamp(MIN, VAL, MAX) 与 max(MIN, min(VAL, MAX)) 是等价的。

乍一看，`clamp` 似乎用途不大，特别是像下面这样的例子：

```CSS
.element {
  font-size: clamp(12px, 16px, 20px);
}
```

在这个例子中，`clamp` 的计算结果是 16px，因为这是个静态值。当首选值是动态的时候，`clamp` 才会大放异彩。其中的一个场景是，使用视口单位 `vw`，其中 `1vw` 是当前视口宽度的 `1%`。如果视口的宽度是 400px 那么 1vw 等价于 4px。只要视口宽度发生变化，浏览器都需要重新计算该值并将其解析为 CSS 像素。这就是我们用线性插值来代替媒体查询的关键因素。

了解其工作原理的最佳方法是将其画成示意图。阅读  Adrian Bece [现代流体排版编辑器](https://modern-fluid-typography.vercel.app/)，可以更好地了解 `clamp` 的工作原理：

![](https://s2.loli.net/2022/03/14/xsajcDlFitQ7P5w.jpg)

最左边的水平线表示 clamp 返回的最小值；最右边的水平线对应最大值。在这两个端点之间是首选值，该值呈线性向上延伸。

因此，如果我们以 vw 单位设置首选值，clamp 将保证它不会超出最小值和最大值的界限，而视口单位的性质将允许该值在这两个值之间线性增大。下面是一个例子：

```CSS
.element {
  font-size: clamp(1rem, 0.45vw + 0.89rem, 1.25rem);
}
```

也就是说，element 的字体至少是 1rem，但也不会超过 1.25rem，它的首选值是 0.45vw + 0.89rem。



上面代码的首选值看起来似乎比较随意，但事实证明，我们要通过一些[数学知识](https://www.aleksandrhovhannisyan.com/blog/fluid-type-scale-with-css-clamp/)来计算出正确的值，只要给定了最小/最大字体。还有很多工具可以帮助我们生成 `clamp` 声明。其中之一就是我自己的[流式类型比例计算器](https://www.fluid-type-scale.com/)，它可以让你在任何项目中添加流式字体大小变量。

![](https://www.aleksandrhovhannisyan.com/assets/images/posts/writing-better-css/fluid-type-scale-2400.webp)

虽然这里的示例是关于字体大小的，但是 `clamp` 可以应用于任何数字属性，包括填充、边距、边框等。我推荐大家在项目中尝试一下 `clamp`，看看它是否适合你们的设计。


尽管这种线性变化看起来很棒，但是如果设计师没有对页面设计这种样式行为​，那么在这种场景下使用它可能并不合适。有时，我们确实只是希望属性值在两个离散状态之间变化，而不是让它做连续的线性变化。在这种情况下，媒体查询依然是我们唯一的选择。总而言之，尽管 `clamp` 非常有用，并且在许多流式布局中有所应用，但它并不能替代媒体查询。


## 使用 gap 简化布局

在 CSS grid 出现之前，能够在 Web 页面上实现动态布局的唯一可行方案就是 Flexbox。但是它在使用过程中，存在一个较大的限制：缺少对间隙的支持。尽管一些设计工具支持间隙的概念，但是 CSS 却不支持间隙，在绝大多数样式中都要用外边距来分隔 flex 子元素。因此，我们经常不得不对最后一个 flex 子元素做一些特殊处理：

```CSS
.flex-item:not(:last-of-type) {
  margin-right: 8px;
  margin-bottom: 8px;
}
```

如果要 flex 布局中的最后一行元素取消底部的外边距，这个功能看似简单，真正实现起来还是挺困难的。除非我们在布局上使用负外边距来抵消这些子元素的外边距。因此，我们不得不写更多的 CSS 来实现这个任务。

现在，有另外一种解决方案了，那就是在 flex 布局中支持使用间隙，也就是 `gap` 属性。如果不考虑一些老版本浏览器的兼容问题，我们可以写出更简单一些的 CSS，如下所示：

```CSS
.flex-layout {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
}
```

除此之外，我们也可以使用 grid 实现布局，它不仅支持 `gap`，而且支持自动适配网格容器中列的尺寸。

```CSS
.grid-layout {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(some-min, 1fr));
  gap: 8px;
}
```

无论使用上面提到的哪种方式，都不必再担心布局的边缘间距，浏览器会自动为我们处理好，只有相邻的子元素之间存在对应的间隙。如果布局不需要换行，那么它的外边缘就不会有任何不必要的间距。此外，由于 flex 和 grid 布局会适配 RTL（右对齐），这使得它们非常适合创建 RTL 安全布局，因为我们不必担心文本的方向性。


## 使用 aspect-ratio 设置宽高比

假如，你想做一个 3 x 3 的正方形九宫格：

![](https://s2.loli.net/2022/02/12/HolrCLXMK8yRUDg.jpg)

在 CSS 还没有 `aspect-ratio` 属性之前，我们通常是使用内边距百分比技巧来实现响应式的正方形：

```CSS
.square {
  height: 0;
  padding-bottom: 100%;
}
```

或者，使用自定义属性来保持元素的宽度和高度同步。这种方法的主要缺点是，由于要设置显式尺寸，因此无法使用响应式尺寸：

```CSS
.square {
  --size: 2rem;
  width: var(--size);
  height: var(--size);
}
```

但现在，所有主流浏览器都支持 `aspect-ratio`，可以用它更直观地表达元素的宽度和高度之间的关系：

```CSS
.square {
  aspect-ratio: 1;
}
```

也许,你决定给正方形一个明确的宽度，让它的高度匹配：

```CSS
.square {
  aspect-ratio: 1;
  width: 2rem;
}
```

或者，使用 grid 布局，并通过网格的格式上下文来设置其宽度：

```CSS
.square-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 1rem;
}
.square {
  aspect-ratio: 1;
}
```

不管是哪种方式，都能实现一个正方形，并能够自动的缩放。

![](https://s2.loli.net/2022/02/12/rwXTxydEnLRj2lD.jpg)

关于 `aspect-ratio`，我最喜欢的将它和其他一些属性组合起来使用，比如下面这个圆形：

```CSS
.circle {
  aspect-ratio: 1;
  border-radius: 50%;
}
```

![](https://s2.loli.net/2022/02/12/c4N9M5mxg237zQL.jpg)

还有很多使用 `aspect-ratio` 的场景值得去探索。一个例子是，根据需要调整嵌入媒体的大小，比如 YouTube 视频；另一个例子是，增强用户体验，当我们使用 HTML 属性来设置图片的宽高，使用 `aspect-ratio` 可以防止布局发生变化。


如果说每分钟前端都会诞生很多新框架和类库，那么 CSS 的变化可能要按年来计算。每年 CSS 会出现一些新特性，包括  [leading-trim](https://medium.com/microsoft-design/leading-trim-the-future-of-digital-typesetting-d082d84b202), [container queries](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Container_Queries), [relative colors](https://blog.jim-nielsen.com/2021/css-relative-colors/) 等一系列让人兴奋的功能。只要对 CSS 知识掌握的越多，就能写出好的代码。



