---
title: React18 用 createRoot 替换 render
date: 2021-08-03 17:05:55
tags: React
---

> 本文源于翻译 [Replacing render with createRoot ](https://github.com/reactwg/react-18/discussions/5)


## 概述

React 18 提供了两个 root API，被称之为 Legacy Root API 和 New Root API：

- Legacy Root API：是指之前版本的 root API `ReactDOM.render`，它将创建一个以 "legacy" 模式运行的 root，其工作方式与 React 17 完全相同。我们会给这个 API 添加一个警告，来说明它将要被弃用并建议切换到 New Root API。

- New Root API：新的 root API 是 `ReactDOM.createRoot`。它可以在 React 18 中创建一个 root，并支持 React 18 中支持的所有新特性。

## 什么是 root?

在 React 中，"root" 是一个指向顶层数据结构的指针，React 用它来跟踪要渲染的树。

在 Legacy Root API 中，root 对用户来说是不透明的，因为我们将它附加到 DOM 元素上，通过 DOM 节点访问它，并没有将其暴露给用户：

```JavaScript
import * as ReactDOM from 'react-dom';
import App from 'App';

const container = document.getElementById('app');

// Initial render.
ReactDOM.render(<App tab="home" />, container);

// During an update, React would access
// the root of the DOM element.
ReactDOM.render(<App tab="profile" />, container);
```

在 New Root API 中，`createRoot` 创建一个 root，然后调用 `render` 方法完成渲染：

```JavaScript
import * as ReactDOM from 'react-dom';
import App from 'App';

const container = document.getElementById('app');

// Create a root.
const root = ReactDOM.createRoot(container);

// Initial render: Render an element to the root.
root.render(<App tab="home" />);

// During an update, there's no need to pass the container again.
root.render(<App tab="profile" />);
```

## 它们的区别是什么?

我们更改这个 API 有以下几个原因。

首先，这修复了 API 在运行更新时的一些人类工程学问题。如上所示，在 Legacy API 中，你需要多次将容器元素传递给 `render`，即使它从未更改过。这也意味着我们不需要将根元素存储在 DOM 节点上，尽管我们今天仍然这样做。

其次，这一变化允许让我们可以移除 `hydrate` 方法并替换为 root 上的一个选项；删除渲染回调，这些回调在部分 hydration 中是没有意义的。

> 译者注：「这一变化允许让我们可以移除 `hydrate` 方法并替换为 root 上的一个选项」这句话的意思是可以这么用 createRoot： createRoot(container, { hydrate: true }).render(<App />)
> 但是值得注意的是，最新的版本中 createRoot 要废弃 `hydrate: true` 这一用法，并引入新的 `hydrateRoot` 支持，具体见 [https://github.com/facebook/react/pull/21687/files](https://github.com/facebook/react/pull/21687/files)。
> 

## 什么是 hydration ？

我们已经把 `hydrate` 函数移到了 `hydrateRoot` API 上。

老版本：

```JavaScript
import * as ReactDOM from 'react-dom';
import App from 'App';

const container = document.getElementById('app');

// Render with hydration.
ReactDOM.hydrate(<App tab="home" />, container);
```

新版本：

```JavaScript
import * as ReactDOM from 'react-dom';

import App from 'App';

const container = document.getElementById('app');

// Create *and* render a root with hydration.
const root = ReactDOM.hydrateRoot(container, <App tab="home" />);
// Unlike with createRoot, you don't need a separate root.render() call here
```

注意，与
createRoot 不同，hydrateRoot 接受原生 JSX 作为第二个参数。这是因为初始客户端渲染是特殊的，需要与服务器树匹配。

如果你想在 hydration 后再次更新 root，你可以将它保存到一个变量中，就像使用 createRoot 一样，然后调用 root.render()：

```JavaScript
import * as ReactDOM from 'react-dom';
import App from 'App';

const container = document.getElementById('app');

// Create *and* render a root with hydration.
const root = ReactDOM.hydrateRoot(container, <App tab="home" />);

// You can later update it.
root.render(<App tab="profile" />);
```
## 什么是渲染回调?

在 Legacy Root API 中，你可以给 `render` 传递一个回调函数，在组件被渲染或更新后调用：

```JavaScript
import * as ReactDOM from 'react-dom';
import App from 'App';

const container = document.getElementById('app');

ReactDOM.render(container, <App tab="home" />, function() {
  // Called after inital render or any update.
  console.log('rendered').
});
```

在 New Root API 中，我们删除了此回调。

对于部分 hydration 和渐进式 SSR，这个回调的时间将不符合用户的期望。为了避免混乱，我们建议在 root 上使用 `requestdlecallback`、`setTimeout` 或 `ref` 回调。

不推荐的写法：

```JavaScript
import * as ReactDOM from 'react-dom';

function App() {
  return (
    <div>
      <h1>Hello World</h1>
    </div>
  );
}

const rootElement = document.getElementById("root");

ReactDOM.render(<App />, rootElement, () => console.log("renderered"));
```

推荐的写法：

```JavaScript
import * as ReactDOM from 'react-dom';

function App({ callback }) {
  // Callback will be called when the div is first created.
  return (
    <div ref={callback}>
      <h1>Hello World</h1>
    </div>
  );
}

const rootElement = document.getElementById("root");

const root = ReactDOM.createRoot(rootElement);
root.render(<App callback={() => console.log("renderered")} />);
```

见 [codesandbox](https://codesandbox.io/s/cold-pine-eyr62?file=/src/index.js)

## 为什么要同时支持两种 API？

在 React 18 中保留 Legacy Root API 有两个原因：

- **平滑升级**：我们希望避免用户升级到 React 18 出现问题。相反，我们向旧的 root API 添加了一个警告，建议使用新 API。

- **实验**：一些应用程序可能会通过实验来比较 Legacy Root API 和 New Root API，其中包括开箱即用的性能改进。在 React 18 同时支持这两种 API，开发人员可以更轻松地对两者做实验对比。