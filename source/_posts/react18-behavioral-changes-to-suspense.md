---
title: React 18 Suspense 的变化
date: 2021-08-18 14:14:21
tags: React
---

> 本文源于翻译 [Behavioral changes to Suspense in React 18](https://github.com/reactwg/react-18/discussions/7)

## 概述

在 React 16.x 版本中，我们基本支持了 Suspense 功能。但是，那时并没有完美支持 Suspense，在我们的示例中有一些东西并未展示出来，比如延迟变化（解析数据完成之后进行状态转换）、占位符节流（限制嵌套和连续的占位符来减少 UI 抖动）和 SuspenseList（调整列表或栅格组件，如按顺序流式处理）等。为了方便做区分，我们把 React 16 和 17 版本中的 Suspense 称为 Legacy Suspense。

我们全套的 Suspense 功能依赖于 Concurrent React，这些功能将会在 React 18 版本里面支持。这意味着 Suspense 在 React 18 中的工作方式与以前的版本会略有不同。从技术上来说，这是一个突破性的变化，但与自动批处理更新一样，预计会对现有代码的影响相对较小，并且不会对应用程序的迁移造成较大的负担。

本文主要讨论 Suspense 的行为差异 —— 影响用户组件代码兼容性的部分。

## 术语说明

**这个功能本身仍然被称为 "Suspense"**

Legacy Suspense 和 Concurrent Suspense 之间的区别只在迁移的背景下才重要。因为我们希望大多数人在升级时不会遇到任何重大障碍，所以在迁移场景之外，我们不会提到这些术语。

## 悬停组件的兄弟组件会被中断

### 简单解释

Legacy Suspense 和 Concurrent Suspense 两者的基本的用户体验是一致的。在下面的代码示例中，组件 `ComponentThatSuspends` 在请求处理数据过程中，React 会在它的位置上展示 `Loading` 组件：

```JavaScript
<Suspense fallback={<Loading />}>
  <ComponentThatSuspends />
  <Sibling />
</Suspense>
```

两者的不同点主要体现在悬停组件 (suspended component) 对其同级组件渲染带来的影响：
- Legacy Suspense 中，同级兄弟组件会立即从 DOM 上卸载（mounted），相关的 effects 和生命周期会被触发，最后会隐藏这个组件。具体可以查看[代码示例](https://codesandbox.io/s/keen-banach-nzut8?file=/src/App.js)。
- Concurrent Suspense 中，同级兄弟组件并不会从 DOM 上卸载，相关的 effects 和生命周期会在 `ComponentThatSuspends` 处理完成时触发。具体可以查看[代码示例](https://codesandbox.io/s/romantic-architecture-ht3qi?file=/src/App.js)。

### 详细解释

在以往的 React 版本里，已经形成这样一个固有印象，当一个组件开始渲染那么就一定会完成渲染。例如，在类组件的渲染过程中，`render` 方法和组件的 `componentDidMount/Update` 生命周期是 1:1 对应的。虽然大多数开发人员并没有真正思考过这个过程，也没有下意识的使用它，但是有可能无意中依赖它而没有意识到。

不难发现，这点对于一些功能是特别重要的，Suspense 的作用就是延迟子组件的渲染，直到组件树依赖的数据解析完成再进行渲染。如果某个组件还没有准备好提交 (commit) 操作，我们该如何处理它的兄弟节点，其中一些可能已经开始渲染了？(例如，如果列表中的第三个组件处于悬停中，而前两个组件的 `render` 方法将被调用。）

当我们第一次引入 Legacy Suspense 时，我们发现了一种保持 1:1 render-commit 对应关系的巧妙方法：我们将跳过悬停的组件，继续渲染兄弟组件，并尽可能多地更新到 DOM。这意味着 DOM 会出现不一致的状态，但我们可以避免这种情况，因为无论如何，我们将用一个 fallback UI 来替换它。在允许浏览器绘制之前，我们将显示 fallback UI，并使用`display:hidden` 隐藏 Suspense 边界内的所有内容。

使用这个小技巧，兄弟节点的渲染行为不受影响，但从用户的角度来看，他们看不到任何不一致：他们只看到一个占位符。

Legacy Suspense 的实现方式虽然听起来有点奇怪，但它却是一个很好的折衷方案，以向后兼容的方式引入了 Suspense 的基本功能。

在 Concurrent Suspense 中，我们所做的是中断兄弟组件并阻止他们提交到 DOM 树。直到相关数据被处理之后，才会将 Suspense 边界内的内容进行提交，这里面包括被悬停的组件和它的兄弟节点。然后将批量处理成一个一致的状态提交到的整个树。无论是从实现的复杂性方面，还是以此为基础支持的功能，这都比较适合我们的渲染模型。从开发人员的角度来看，这可以说是一种更可预测的行为，因为副作用不应该渲染在页面上（这已经被阻止了）。

因此，需要让我们的代码能够支持这种中断。这与使用 [startTransition](https://github.com/reactwg/react-18/discussions/41) 有着同样的要求。通常，这些实现中都涉及将副作用 (effect) 和突变 (mutation) 从渲染阶段移动到提交阶段。可以使用 Strict Mode 在开发过程中尽早发现这些类型的 bug。

## Suspense 边界之外的 ref

另一个差异实际上也是 render-commit 问题引起的：父级 ref 传入的时间。

```JavaScript
const refPassedFromParent = useRef(null)
<Suspense fallback={<Loading />}>
  <ComponentThatSuspends />
  <button ref={refPassedFromParent} {...buttonProps} />
</Suspense>
```

在 Legacy Suspense 中，在渲染之初 `refPassedFromParent.current` 立即指向 DOM 节点，此时 `ComponentThatSuspends` 还未处理完成。

在 Concurrent Suspense 中，在 `ComponentThatSuspends` 完成处理、Suspense 边界解除锁定之前 `refPassedFromParent.current` 一直为 null。

也就是说，在父级代码中访问此类 ref 都需要关注当前 ref 是否已经指向相应的节点。

我们认为这导致行为出现差异的可能性很低，事实上，新的行为与 React 的其它的渲染模型更加一致。但值得注意的是，它可能会影响现有代码。
