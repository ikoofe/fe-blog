---
title: React 18 严格模式支持 Strict Effects
date: 2021-08-09 06:12:00
tags:
---


> 本文来源于翻译文章 [Adding Strict Effects to StrictMode](https://github.com/reactwg/react-18/discussions/19)

## 概述

严格模式（[Strict Mode](https://reactjs.org/docs/strict-mode.html)）从 React 16.3 版本开始支持，用于标记出应用中潜在的代码问题。在 React 应用中添加 `<StrictMode>`，会对其内部的组件开启一些特殊的行为，并且这些行为只在开发模式下有效。例如，在严格模式下，React 会故意加倍渲染组件，以消除不安全的副作用。

随着 React 18 的发布，StrictMode 增加了对 Strict Effects 的支持。在严格模式下，**React 会对新安装的组件调用两次 effect（`mount` -> `unmount` -> `mount`）**。与其他严格模式下的行为一样，React 仅在开发环境中执行此操作。

## 为什么 React 增加 Strict Effects？

React 的众多功能都会面临着一个约束条件：组件不只一次的挂载 (mount) 和卸载 (unmount)。

Fast Refresh 就是这些功能中的一个，在 Next.js、Create React App 和 React Native 中它都默认被开启，每当修改并保持文件时，effect 都会被重新执行。这意味着，如果某个组件或库偶尔由于重新运行其 effect 时出现问题，它将无法很好地使用 Fast Refresh。

在我们将要支持的功能中同样依赖于这个约束。例如，我们正在开发一种新的 Offscreen API，它将使我们能够更好地支持 UI，如选项卡式容器和虚拟化列表，并更好地使用新的浏览器 API，如 content-visibility（它还将有助于 React Native 团队正在进行的渲染前优化）。但要达到这一目标，我们需要对 effect 的工作方式做一些调整。

假设您有一个按条件渲染的组件，例如 current 选项卡。如果该组件或它的子组件中有一个状态 (state)，但是当组件一旦被卸载 (unmount) 这个状态将会立即被丢失。到目前为止，保存状态的唯一方法是将其 “提升” 到更高的组件（或像 Redux 这样的外部存储）中。

新的 Offscreen API（以及本文中描述的 effect 调整）的主要目的是允许 React 通过隐藏而不是卸载组件的方式来保持这些状态不会被丢失。在执行此操作时，React 将调用与卸载时相同的生命周期 hooks，但它还将同时保留 React 组件和 DOM 元素的状态。

也就是说，**组件可以多次 “挂载” 和 “卸载”**。

当一个组件被隐藏时，它无论是在选项卡式容器中还是在虚拟化列表中，它在 DOM 中不再可见，就像它从 DOM 中卸载（删除）一样。实际上它并没有被卸载（因为我们希望保留状态），但是从用户的角度来看，它们呈现的效果是相同的。

对于已卸载的组件来说，触发一些命令性代码（例如定位工具提示）是没有意义的。对于隐藏的组件也是如此。所以 React 需要告诉组件它已经被隐藏。那么如何实现呢？答案是将组件卸载时调用的函数（effect 清理函数或 `componentWillUnmount`）调用一次。

当一个组件在隐藏后再次显示时，这个过程与再次挂载类似，除了它保留了以前的所有状态。React 调用它在挂载时调用的函数（effect 或 `componentDidMount`）告诉组件再次显示出来。

我们发现，**这种方法在大多数产品代码都能够 “正常工作”，但有些调整是必要的**。

> 译者注，在留言区 React 开发人员对这句话做了一些补充说明，以方便大家更好的理解这句话：“... 我们在数千个组件上推出了 Strict Effects，几乎没有碰到什么问题。在一些非常复杂的库代码中，我们确实遇到的一些障碍，这些代码大量的进行手动 DOM 操作。这就是为什么我们相信，可以将这种行为添加到严格模式中，而不会给开发人员带来不必要的负担。几乎所有的事情都能正常工作 (just work) ...”

## 关闭 Strict Effects

如果 Strict Effects 会给你的应用程序带来了严重问题，可以把 `<StrictMode>` 禁用了，直到把问题修复为止。当前无法保留旧的 `<StrictMode>` 行为，一旦启用它，它就会包含 Strict Effects 功能。
