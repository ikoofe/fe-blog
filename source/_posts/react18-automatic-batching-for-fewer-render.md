---
title: React 18 批量更新减少渲染次数
date: 2021-06-25 14:37:55
tags: React
---


> 本文源于翻译 [Automatic batching for fewer renders in React 18 ](https://github.com/reactwg/react-18/discussions/21)，由新东方在线前端工程师 **李方** 翻译。

# React 18 批量更新减少渲染次数

## 概述

React 18 增加了一个新的优化特性，在代码中无需手动处理，就可以支持更多场景下的批量更新 (batching)。本文将说明什么是批量更新，在 React 18 版本以前它是如何工作的，以及它在 React 18 版本发生了怎样的变化。

## 什么是批量更新？

批量更新是指 React 将多次 state 更新进行合并处理，最终只进行一次渲染，以获得更好的性能。

例如，如果在同一个点击事件中有两个状态更新，React 总是会把它们批量处理成一个重新渲染。 如果运行以下代码，我们会看到每次点击时，虽然设置了两次状态，React 也只执行一次渲染：

```JSX
function App() {
  const [count, setCount] = useState(0);
  const [flag, setFlag] = useState(false);

  function handleClick() {
    setCount(c => c + 1); // Does not re-render yet
    setFlag(f => !f); // Does not re-render yet
    // React will only re-render once at the end (that's batching!)
  }

  return (
    <div>
      <button onClick={handleClick}>Next</button>
      <h1 style={{ color: flag ? "blue" : "black" }}>{count}</h1>
    </div>
  );
}
```

- ✅ [Demo：React 17 中事件处理函数中的批量更新](https://codesandbox.io/s/spring-water-929i6?file=/src/index.js)（ 注意每次点击在控制台中有一次打印 ）

批量更新可以提高组件的渲染性能，因为它避免了不必要的渲染。同时，也防止了组件中只更新一个状态变量，导致组件其他状态变化并未完全渲染出来，这可能会引起 bug。与餐馆点菜的情景类似，服务员不会在我们点第一道菜时就到厨房下单，而是等点单完成才会一起下单。

然而，React 的批量更新并不是所有场景都会生效。例如，如果在 `handleClick` 中请求数据，然后在数据请求成功之后更新状态，那么 React 不会触发批量更新，而是执行两次独立的更新。

这是因为，之前版本的 React 要求只有在浏览器事件（如点击事件）中才会触发批量更新。但是，在下面的代码示例中，当数据请求成功之后，点击事件早已经结束了，这时进行状态更新（在 fetch 回调函数中）是不会触发批量更新的：

```JSX
unction App() {
  const [count, setCount] = useState(0);
  const [flag, setFlag] = useState(false);

  function handleClick() {
    fetchSomething().then(() => {
      // React 17 and earlier does NOT batch these because
      // they run *after* the event in a callback, not *during* it
      setCount(c => c + 1); // Causes a re-render
      setFlag(f => !f); // Causes a re-render
    });
  }

  return (
    <div>
      <button onClick={handleClick}>Next</button>
      <h1 style={{ color: flag ? "blue" : "black" }}>{count}</h1>
    </div>
  );
}
```

- 🟡 [Demo：React 17 不会批处理外部事件处理函数](https://codesandbox.io/s/trusting-khayyam-cn5ct?file=/src/index.js)（ 注意每次点击在控制台中会有两次打印 ）

在 React 18 之前，我们只在 React 事件处理函数中执行过程中进行批量更新。在默认情况下，对 `promises`、`setTimeout`、原生事件处理函数或其他任何事件中的状态更新都不会进行批量更新。

## 什么是自动批量更新？

从 React 18 的 `createRoot` 开始，不论在哪里， 所有更新都将自动进行批量更新。

这意味着 `setTimeout`、`promises`、原生事件处理函数或其他任何事件的批量更新都将与 React 事件一样，以相同的方式进行批量更新。我们希望这样可以减少渲染工作量，从而提高应用程序的性能:

```JSX
function App() {
  const [count, setCount] = useState(0);
  const [flag, setFlag] = useState(false);

  function handleClick() {
    fetchSomething().then(() => {
      // React 18 and later DOES batch these:
      setCount(c => c + 1);
      setFlag(f => !f);
      // React will only re-render once at the end (that's batching!)
    });
  }

  return (
    <div>
      <button onClick={handleClick}>Next</button>
      <h1 style={{ color: flag ? "blue" : "black" }}>{count}</h1>
    </div>
  );
}
```
- ✅ [Demo：React 18 使用 `createRoot` 对外部事件处理程序进行批量处理](https://codesandbox.io/s/morning-sun-lgz88?file=/src/index.js)（ 注意每次点击在控制台中有一次打印 ）
- 🟡[ Demo：React 18 传统的 `render` 保留之前的行为](https://codesandbox.io/s/jolly-benz-hb1zx?file=/src/index.js)（ 注意每次点击在控制台中会有两次打印 ）

> 注意：希望你升级到 React 18 并使用其中的 `createRoot`。旧的 `render` 仅仅是为了简化两个版本的生产实验。

无论状态在哪里发生变化，React 都会进行批量更新，像这样：

```JSX
function handleClick() {
  setCount(c => c + 1);
  setFlag(f => !f);
  // React will only re-render once at the end (that's batching!)
}
```

或者像这样：

```JSX
setTimeout(() => {
  setCount(c => c + 1);
  setFlag(f => !f);
  // React will only re-render once at the end (that's batching!)
}, 1000);
```

或者像这样：

```JSX
fetch(/*...*/).then(() => {
  setCount(c => c + 1);
  setFlag(f => !f);
  // React will only re-render once at the end (that's batching!)
})
```

或者像这样：

```JSX
elm.addEventListener('click', () => {
  setCount(c => c + 1);
  setFlag(f => !f);
  // React will only re-render once at the end (that's batching!)
});
```

> 注意：只有在通常安全的情况下 React 才会执行批量更新。例如，React 需要确保对于每个用户发起的事件（如点击或按键），DOM 在下一个事件之前完全更新。再例如，这可以禁止提交时禁用的表单被提交两次。

## 如果不想批量更新怎么办？

通常批处理是安全的，但有些代码可能依赖于在状态更改后立即从 DOM 中读取某些内容。 对于这种情况，可以使用 `ReactDOM.flushSync()` 选择不进行批量处理：

```JSX
import { flushSync } from 'react-dom'; // Note: react-dom, not react

function handleClick() {
  flushSync(() => {
    setCounter(c => c + 1);
  });
  // React has updated the DOM by now
  flushSync(() => {
    setFlag(f => !f);
  });
  // React has updated the DOM by now
}
```
这种场景应该不会经常出现。

## 这对 Hooks 有什么影响吗？

如果你正在使用 Hooks，在绝大多数情况下批量更新都能“正常工作”。

## 这对 Classes 有什么影响吗？

React 在执行事件回调期间的状态更新一直都是批量处理的，所以对于这些更新不会有任何变化。

在类组件中有一个比较极端的情况，可能会引起问题。

类组件有一个要特别的注意的现象，它可以同步读取事件内部的状态更新。也就是说，能够在两次调用 `setState` 之间读取 `this.state`：

```JSX
handleClick = () => {
  setTimeout(() => {
    this.setState(({ count }) => ({ count: count + 1 }));

    // { count: 1, flag: false }
    console.log(this.state);

    this.setState(({ flag }) => ({ flag: !flag }));
  });
};
```

在 React 18 中，不会出现上面提到的现象。 因为在 `setTimeout` 中的所有状态更新都是进行批量处理的，因此 React 不会同步渲染第一个 `setState` 的结果 —— 渲染将发生在下一个浏览器周期中。 所以渲染还没有发生：

```js
handleClick = () => {
  setTimeout(() => {
    this.setState(({ count }) => ({ count: count + 1 }));

    // { count: 0, flag: false }
    console.log(this.state);

    this.setState(({ flag }) => ({ flag: !flag }));
  });
};
```
见 [sandbox](https://codesandbox.io/s/interesting-rain-hkjqw?file=/src/App.js)

如果这个问题阻碍了升级到 React 18，可以使用 `ReactDOM.flushSync` 强制更新，但建议谨慎使用：

```JSX
handleClick = () => {
  setTimeout(() => {
    ReactDOM.flushSync(() => {
      this.setState(({ count }) => ({ count: count + 1 }));
    });

    // { count: 1, flag: false }
    console.log(this.state);

    this.setState(({ flag }) => ({ flag: !flag }));
  });
};
```

见 [sandbox](https://codesandbox.io/s/hopeful-minsky-99m7u?file=/src/App.js)

这个问题对 Hooks 函数组件没有影响，因为设置 state 不会更新 `useState` 中的现有变量：

```js
function handleClick() {
  setTimeout(() => {
    console.log(count); // 0
    setCount(c => c + 1);
    setCount(c => c + 1);
    setCount(c => c + 1);
    console.log(count); // 0
  }, 1000)
```

在采用 Hooks 函数组件中，不用做任何处理，它已经为批量更新铺平了道路。

## `unstable_batchedUpdates` 是什么？

一些 React 库会使用这个 API 强制对事件处理之外的 `setState` 进行批量更新：

```JSX
import { unstable_batchedUpdates } from 'react-dom';

unstable_batchedUpdates(() => {
  setCount(c => c + 1);
  setFlag(f => !f);
});
```

这个 API 在 18 中仍然存在，但不再需要它了，因为批量处理是自动进行的。我们没有在 18 版本中删除它，当主流库不再依赖于它之后，在未来的某个版本会删除它。

> 译者注：React 的批量更新，可以理解为在 18 版本之前是个半成品，只能在特定场景（事件回调函数执行期间的状态更新）才会自动触发，其他场景只能借助 `unstable_batchedUpdates` 来实现批量更新。而在 18 版本更加通用了，但是也带来一个问题，类组件不再支持某些场景下的同步状态更新，需要调用 `flushSync` 来更新状态。`flushSync` 可将关闭批量更新。