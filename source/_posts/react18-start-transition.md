---
title: React18 新特性：startTransition
date: 2021-08-12 16:50:56
tags: React
---

> 本文源于翻译 [New feature: startTransition ](https://github.com/reactwg/react-18/discussions/41)

## 概述

在 React 18 中引入了一个新的 API，它可以帮助我们的应用程序保持较高的响应，即便是在页面上有大量更新的时候。这个新的 API 支持我们把特定的更新标记为 "transition"，进而来改进用户交互效果。 React 将允许我们在状态转换期间提供视觉反馈，并在转换发生时保持浏览器的响应。

[案例：为慢速渲染添加 startTransition](https://github.com/reactwg/react-18/discussions/65)

## 这能解决什么问题？

构建体验流畅、响应快速的应用程序并不是一件容易的事情。 有时候，像按钮点击或输入框输入文本这样的较小的操作，可能会引起页面上发生很多事情。当所有的这些动作都在进行时，很可能会导致页面冻结或挂起。

例如，在一个筛选数据的输入框中输入字符时，需要将输入的文本存储在 state 中，以便我们进行数据列表中筛选出相关数据和在输入框中展示该文本。代码如下所示：

```JavaScript
// Update the input value and search results
setSearchQuery(input);
```

此时，每当用户输入一个字符时，我们都会更新输入值，并使用新值来搜索列表并显示结果。对于屏幕上有大量更新时，这可能会导致页面在渲染内容时出现延迟，从而使打字或其他交互感觉缓慢或者卡顿。即便列表的数据不是太多，列表项本身也可能很复杂，并且每次按键都会有不同的结果，所以并没有明确的方法来对它们的渲染进行优化。

从概念上讲，针对这个问题可以将更新分为两个。第一个是紧急更新（urgent update），用于更改输入字段的值，可能还会更改其周围的一些 UI。第二个是非紧急更新，用于显示列表数据的筛选结果。

```JavaScript
// Urgent: Show what was typed
setInputValue(input);

// Not urgent: Show the results
setSearchQuery(input);
```

在上面的代码中，用户期望第一个更新是即时的，因为本地浏览器处理这些交互的速度很快。但是第二个更新可能会有点延迟，但是用户也并不期望它立即完成，因为用户可能有很多工作要做（比如，继续输入字符等）。实际上，开发人员经常使用 debounce（防抖）等技术人为地延迟此类更新。

在 React 18 之前，所有更新都是紧急更新。这意味着上面的两个状态会被同时渲染，并且在渲染完成之前会阻塞用户看到交互的反馈结果。因此，我们缺少的是一种方法来告诉 React 哪些是紧急更新，哪些是非紧急更新。

## startTransition 有什么帮助？

新的 `startTransition` API 通过把更新标记为 "transition" 来解决这个问题：

```JavaScript
import { startTransition } from 'react';


// Urgent: Show what was typed
setInputValue(input);

// Mark any state updates inside as transitions
startTransition(() => {
  // Transition: Show the results
  setSearchQuery(input);
});
```

在 `startTransition` 中的更新被视为非紧急更新，如果出现更紧急的更新（如点击或按键），更新会被中断。如果用户中断了 `startTransition` 中的更新（例如，连续输入多个字符），React 将丢掉未完成的过时的渲染工作，只渲染最新的更新。

过渡更新 (transition) 可让你保持大多数交互的敏捷性，即使它们导致显著的 UI 变化。 它们还可以让你避免在渲染不相关的内容时浪费时间。

## 什么是过渡更新？

我们将状态更新分为两类：

- **紧急更新 (Urgent updates)** 反映了直接交互，如打字、点击、按键等。
- **过渡更新 (Transition updates)** 将 UI 从一个视图转换到另一个视图。

输入、点击或按键等紧急更新需要立即响应，以符合我们对物体行为的直觉。否则他们会觉得 “错了”。然而，过渡是不同的，因为用户不希望在屏幕上看到每个中间值。

例如，当我们在下拉列表中选择一个筛选项时，我们期望筛选按钮本身在单击时要立即响应。但是，实际筛选结果可以分别过渡，因为一个小的延迟是难以察觉的，而且通常是预料之中的。如果在结果渲染完成之前再次更改了选项，我们只关心看到的是最新的结果。

在典型的 React 应用中，大多数更新在概念上都是过渡更新。但出于向后兼容性的原因，所以过渡更新是可选的。默认情况下，React 18 仍然将更新处理为紧急更新，你可以把更新包裹在 `startTransition` 中来将更新标记为过渡。

## 它与 setTimeout 有何不同？

上述问题的一个常见解决方案是将第二个更新包裹在 `setTimeout` 中：

```JavaScript
// Show what you typed
setInputValue(input);

// Show the results
setTimeout(() => {
  setSearchQuery(input);
}, 0);
```

这会延迟第二个更新，直到第一个更新之后。 throttling（节流）和 debouncing （防抖）是这种技术的常见变体。

一个重要的区别是 `startTransition` 不像 `setTimeout` 那样在稍后执行，它是立即执行。传递给 `startTransition` 的函数同步运行，但函数内部的任何更新都标记为 "transitions"。React 将在稍后处理更新时使用该信息来决定如何渲染更新。这意味着我们开始渲染更新的时间比在定时器中包裹的更新的时间要早。在一个网速较快的设备上，两次更新之间的延迟非常小。在一个网速较慢的设备上，延迟会更大，但 UI 会依然保持响应。

另一个重要的区别是 `setTimeout` 内的存在较大更新时仍然会锁定页面，只不过是在定时器执行之后。当定时触发时，如果用户仍在输入或与页面交互，它们仍将被阻止与页面交互。但是用 `startTransition` 标记的状态更新是可以中断的，所以它们不会锁定页面。它们让浏览器在渲染不同组件之间的小间隙中处理事件。如果用户输入发生变化，React 将不必继续渲染用户不再感兴趣的内容。

最后，因为 `setTimeout` 只是延迟更新，显示加载中状态指示器需要编写异步代码，这通常是脆弱的。使用过渡更新，React 可以为你跟踪挂起状态，根据过渡的当前状态更新它，并且让你能够在用户等待时显示加载中的状态。

## 在过渡期间我该做什么？

作为一种最佳实践，你需要通知用户后台有工作正在进行。 为此，我们为过渡更新提供了一个带有 `isPending` 标志的 Hook：

```JavaScript

import { useTransition } from 'react';


const [isPending, startTransition] = useTransition();
```

当过渡被挂起时， `isPending` 值为 true，允许你在用户等待时显示加载中状态提示：

```JavaScript
{isPending && <Spinner />}
```

在 `startTransition` 里面的状态更新不必源自同一个组件。例如，搜索输入中的加载中状态提示可以反映重新渲染搜索结果的进度。

## 为什么不写更快的代码？

编写更快的代码并且避免不必要的重新渲染仍然是优化性能的好方法。这与过渡更新是相辅相成的。即使发生了重大的视觉变化，它们也能保持 UI 较高的响应 - 例如，在显示页面上显示新内容时。这很难用现有的策略进行优化。即使已经优化了不必要的重新渲染，与把每个更新都视为紧急更新相比，过渡更新仍然提供了更好的用户体验。

## 我可以在哪里使用它？

您可以在 `startTransition` 中来操作任何可以放在后台的更新。通常，这些类型的更新分为两类：

- 缓慢渲染：这些更新需要时间，因为 React 需要执行大量工作才能改变 UI 以显示结果。[这是添加了 `startTransition` 的真实事例，保持了应用程序在昂贵的重新渲染中的响应。](https://github.com/reactwg/react-18/discussions/65)
- 慢速网络：这些更新需要时间，因为 React 正在等待来自网络的一些数据。这种情况与 Suspense 紧密相关。


