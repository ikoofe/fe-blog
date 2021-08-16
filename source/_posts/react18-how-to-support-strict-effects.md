---
title: React 18 如何支持 Strict Effects
date: 2021-08-09 11:42:33
tags:
---

> 本文来源于翻译文章 [How to support strict effects](https://github.com/reactwg/react-18/discussions/18)

## 概述

*如果你还没有阅读上一篇关于 StrictMode 变化的文章，可以点击链接查看*

首先，让我们看一个组件代码示例：

```JSX
function ExampleComponent(props) {
  useEffect(() => {
    // Effect setup code...

    return () => {
      // Effect cleanup code...
    };
  }, []);

  useLayoutEffect(() => {
    // Layout effect setup code...

    return () => {
      // Layout effect cleanup code...
    };
  }, []);

  // Render stuff...
}
```

这个组件在 mount 和 unmount 时会执行一些 effect，通常这些 effect 只被执行一次（在组件挂载之后），清除函数也只被执行一次（在组件卸载之后）。在 Strict Effects 模式，会触发以下流程：

- React 渲染组件
- React 挂载组件
  - 执行 layout effect 启动逻辑代码
  - 执行 effect 启动逻辑代码
- React 模拟组件被隐藏或卸载
  - 执行 layout effect 清除逻辑代码
  - 执行 effect 清除逻辑代码
- React 模拟组件重新显示或挂载
  - 执行 layout effect 启动逻辑代码
  - 执行 effect 启动逻辑代码

只要一个 effect 在其自身之后进行清理（必要时返回一个清除函数），这通常不会导致问题。大多数 effect 至少有一个依赖项。因此，它们已经能够适应重新挂载多次，并且可以不需要做任何更改。

不过为了多次挂载（卸载）之后组件正常运行，如果 effect 只在挂载时才运行，那么可能需要对 effect 做一些更改。最有可能因为挂载多次而受到影响，导致需要修改 effect 的情况分为以下两类：

- 在卸载时需要执行清理操作的 effect
- 只执行一次的 effect（挂载时或依赖变化时执行）

## Effect 清理操作是对称的

无论是添加监听事件还是某些命令式 API，一般来说，如果 effect 返回了一个清除函数，那么它应该与启动函数是对应的。当前，大部分组件都是使用下面的示例模式：

```JSX
// A Ref (or Memo) is used to init and cache some imperative API.
const ref = useRef(null);
  if (ref.current === null) {
  ref.current = new SomeImperativeThing();
}

// Note this could be useLayoutEffect too; same pattern.
useEffect(() => {
  const someImperativeThing = ref.current;
  return () => {
    // And an unmount effect (or layout effect) is used to destroy it.
    someImperativeThing.destroy();
  };
}, []);
```



如果上面的组件被卸载并重新挂载，那么命令式的 API （这里是指 `ref`）很可能会被破坏(毕竟，它在第一次卸载后就被销毁了）。要解决这个问题，我们需要在再次挂载时（重新）初始化 `ref`。

```JSX
// Don't use a Ref to initialize SomeImperativeThing!

useEffect(() => {
  // Initialize an imperative API inside of the same effect that destroys it.
  // This way it will be recreated if the component gets remounted.
  const someImperativeThing = new SomeImperativeThing();

  return () => {
    someImperativeThing.destroy();
  };
}, []);
```

有时，其他函数（如事件处理）也需要以命令的形式进行交互。在这种情况下，可以使用 ref 来共享该值。

```JSX
// Use a Ref to hold the value, but initialize it in an effect.
const ref = useRef(null);

useEffect(() => {
  // Initialize an imperative API inside of the same effect that destroys it.
  // This way it will be recreated if the component gets remounted.
  const someImperativeThing = ref.current = new SomeImperativeThing();

  return () => {
    someImperativeThing.destroy();
  };
}, []);

const handeThing = (event) => {
  const someImperativeThing = ref.current;
  // Now we can call methods on the imperative API...
};
```

在有些场景中可能需要与其他组件共享命令式 API。在这种情况下，可以使用惰性初始化函数暴露出相关的 API。

```JSX
// This ref holds the imperative thing.
// It should only be referenced by the current component.
const ref = useRef(null);

// This lazy init function ref can be shared with other components,
// although it should only be called from an effect or an event handler.
// It should not be called during render.
const getterRef = useRef(() => {
  if (ref.current === null) {
    ref.current = new SomeImperativeThing();
  }
  return ref.current;
});

useEffect(() => {
  // This component doesn't need to (re)create the imperative API.
  // Any code that needs it will do this automatically by calling the getter.

  return () => {
    // It's possible that nothing called the getter function,
    // in which case we don't have to cleanup the imperative code.
    if (ref.current !== null) {
      ref.current.destroy();
      ref.current = null;
    }
  };
}, []);

```

## 只执行一次的 effect 可以使用 ref

如果 effect 没有清理函数，不需要任何更改即可支持新的特性。让我们来看一个将日志发送到服务器的 effect。

```JSX
useEffect(() => {
  SomeTrackingAPI.logImpression();
}, []);
```

这个 effect 的作用是在用户浏览了特定的内容时要做一下日志记录。但是，当该内容被隐藏并且再展示给用户时，这个过程要如何处理呢？是否应该再发送一次日志呢？（这个场景与 tabs 标签页的切换类似）如果需要再次发送日志，那么就不需要更改 effect。如果要求只发送一次日志，那么我们应该使用 ref，改动之后的代码如下：

```JSX
const didLogRef = useRef(false);

useEffect(() => {
  // In this case, whether we are mounting or remounting,
  // we use a ref so that we only log an impression once.
  if (didLogRef.current === false) {
    didLogRef.current = true;

    SomeTrackingAPI.logImpression();
  }
}, []);
```

## "auto focus" 然后恢复 focus

这里介绍一种比较有趣的业务场景：当点击一个按钮时会打开一个弹窗，并自动 focus 到弹窗的某个元素上；当关闭弹窗时，要重新 focus 到之前触发弹窗的按钮上 (也就是恢复到打开弹窗之前的 focus 状态)。在下面的代码中，`restoreFocus` 记录要恢复 focus 状态的元素。

```JSX
function Modal({ children }) {
  const restoreFocus = React.useRef(null);

  function handleFocus(event) {
    if (restoreFocus.current === null) {
      restoreFocus.current = event.relatedTarget;
    }
  }

  React.useEffect(() => {
    return () => {
      if (restoreFocus.current !== null) {
        restoreFocus.current.focus();
        restoreFocus.current = null;
      }
    };
  }, []);

  return <div onFocus={handleFocus}>{children}</div>;
}
```

在下面的代码中通过按钮控制弹窗的显示和隐藏。正常情况下，打开弹窗后，由于 `<input>`  设置了 `autoFocus` 属性会自动聚焦。进而触发了 Modal 中的 `handleFocus`，并将`<button>` 元素记录到 `restoreFocus`。

```JSX
  const [open, setOpen] = React.useState(false);

  return (
    <React.Fragment>
      <button onClick={() => setOpen(!open)}>{open ? "close" : "open"}</button>
      {open && (
        <Modal>
          <input autoFocus />
        </Modal>
      )}
    </React.Fragment>
  );
```

但是，在 Strict Effects 模式下会通过元素的隐藏和展示来模仿卸载和重新挂载这个过程时，所以当卸载时会聚焦到 `<button>` 元素上，由于重新挂载时元素只是重新显示，这时不会再聚焦到 `<input>` 。所以最终看到的结果与上面提到的情况完全不同。

可以通过如下方式实现自动聚焦：

```JSX
  const [open, setOpen] = React.useState(false);
  const target = React.useRef(null);

  React.useEffect(() => {
    target.current.focus();
  }, []);

  return (
    <React.Fragment>
      <button onClick={() => setOpen(!open)}>{open ? "close" : "open"}</button>
      {open && (
        <Modal>
          <input ref={target} />
        </Modal>
      )}
    </React.Fragment>
  );
```


## 上面的示例并未覆盖所有情形

本文只涵盖了一些最常见的场景，但并不是详尽的列表。我们计划在将来写一些不太常见的案例。同时，如果您不确定 effect 是否应该运行多次，或者 effect 与上面场景不匹配，请咨询我们，我们将提供帮助。
