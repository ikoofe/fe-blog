---
title: React 18 服务端升级
date: 2021-08-23 05:36:08
tags: React
---

> 本文源于翻译 [Upgrading to React 18 on the server](https://github.com/reactwg/react-18/discussions/22)

## 概述

服务端升级到 React 18 版本的方法如下：

- 安装 React 18 最新版本
- 将 `renderToString` 改为 `pipeToNodeWritable` 解锁新功能

React 18 改进了 SSR 的体系结构以提升性能。作为这几年的工作成果，这些改进是非常有用的。

## API 变化

一直以来，React 在服务端上并不支持 Suspense。这在 React 18 中有所改变，选择不同的 API 来支持不同级别的 Suspense：

- `renderToString`: 能够运行 (有限的支持 Suspense)
- `renderToNodeStream`: 不建议使用 (完全支持 Suspense, 不包括 stream)
- `pipeToNodeWritable`: **最新推荐** (完全支持 Suspense 和 stream)


### 已有 API: `renderToString(React.Node): string`

这个 API 能够继续被使用，但它不支持新的功能，所以我们推荐切换到 `pipeToNodeWritable`。由于没有被废弃，我们也可以继续使用 `renderToString`。

在 React 18 中，`renderToString` 非常有限的支持 `<Suspense>` 功能。在此之前，使用 `<Suspense>` 会抛出一个错误。从 React 18 版本开始，我们会将 Suspense 边界标记为 "client-rendered"，并立即触发渲染 `fallback` 中的 HTML。然后，在客户端加载完 JS 之后，会重新渲染它的内容。也就是说，如果在应用的最外层使用了 <Suspense> 并在渲染阶段被悬停 (suspend)，将导致应用程序选择退出服务端渲染。

这个变化并不影响现有的应用程序，因为以前在服务器上 Suspense 根本不起作用。但是，如果是以 条件悬停的方式在客户端上渲染 `<Suspense>`，可能会出现因为条件不匹配，导致它的内容从 DOM 中删除。在服务端和客户端上，首次渲染时通过条件分别渲染不同的内容，这在 React 上是不支持的。

> 注意：这种行为不是很有用，但它已经是我们能做到的最好的了，因为 `renderToString` 是同步的。它不能 “等待” 任何东西。这就是为什么我们推荐使用新的 `pipetonodewriteable`。

### 废弃 API: `renderToNodeStream(React.Node): Readable`

在 React 18 中，完全不推荐使用 `renderToNodeStream`，使用它会发出警告。这是我们添加的第一个流式 API，但它的功能非常不足（无法等待数据）。大家也不常用它。它在 React 18 版本中能够工作，包括下面提到的 Suspense 新功能，但是它将缓冲整个内容直到 stream 结束。换句话说，它不再进行 stream 处理操作。使得它的用途比较让人困惑，这就是不建议使用它的原因。

我们正在用 `pipeToNodeWritable` 代替它。

### 推荐 API: `pipeToNodeWritable(React.Node, Writable, Options): Controls`

这是我们今后推荐的 API，它支持所有的新功能：

- 完全内置支持 <Suspense> （集成数据获取功能）
- 使用 `lazy` 做代码分割，不会出现内容 “消失” 带来的闪屏
- HTML stream 中的延迟内容会在稍后展现出来

在最新的 Alpha 版本中，可以这样使用它：

```JavaScript
import { pipeToNodeWritable } from 'react-dom/server';
```

与 `renderToString()` 不同的是，`pipeToNodeWritable()` 需要更多的代码配置。
我们准备了一个示例，演示[如何将代码从使用renderToString() 更改为pipeToNodeWritable()](https://codesandbox.io/s/festive-star-9hfqt?file=/server/render.js:1043-1575)。之后我们将会提供更详细的文档，我们可以先通过上面的示例来一探究竟。

**如果想更多了解这个 API 解锁的内容，可以阅读 [New SSR Suspense Architecture](https://github.com/reactwg/react-18/discussions/37)。**

这个 API 尚未与获取数据 (data fetching) 集成。Suspense 常用的机制都可以正常支持和工作。但是，关于数据从服务器传输到客户端如何进行预填充缓存，当前我们还没有相关的建议。我们希望在未来能够提供更多的指导。

还有一些问题需要大家进行尝试。例如，如何处理 `<title>` 标签。因为这是一个真正的流式 API，但是在流处理开始时，可能还没有 “理想” 的标题。大家可以去尝试很多解决方案，但我们很好奇的是，当大家在应用程序中使用这个 API 并尝试不同的方法时，最终是否找到了最适合的解决方案。


### 其他 API

还有几个相关的 API 用于产生静态标记。它们的功能如下：

- `renderToStaticMarkup`: 继续使用（有限的支持 Suspense）
- `renderToStaticNodeStream`: 不建议使用（完全支持 Suspense, 不包括 stream）

尚未对它们添加与 `pipeToNodeWritable` 对应的推荐的新 API。
