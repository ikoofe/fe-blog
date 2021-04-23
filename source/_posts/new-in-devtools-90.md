---
title: Chrome 90 DevTools 中的新特性
date: 2021-04-18 11:57:55
tags: devtools
---

> Chrome 90 版本已经发布了一段时间，随之而来的 DevTools 中也新加了一些新特性。本文是对官方文档 [What's New In DevTools (Chrome 90)](https://developer.chrome.com/blog/new-in-devtools-90/) 做了一些简单的翻译。


## CSS flexbox 调试工具

DevTools 有了全新的 CSS flexbox 调试工具。

![](https://developer-chrome-com.imgix.net/image/BrQidfK9jaQyIHwdw91aVpkPiib2/hbg2toNQJqIWB30Mo2xt.png?auto=format&w=1428)


当 HTML 元素的 CSS 样式被设置为 `display: flex` 或 `display: inline-flex` 的时候，在 **Elements** 面板中该元素的后面会有一个 `flex` 标记，当点击这个标记后，在页面上会展示 flex 布局浮层。在 **Styles** 面板中，点击 `display: flex` 或 `display: inline-flex` 后面的图标，可以打开 **Flexbox** 编辑器，可以快速的编辑 flexbox 属性。

另外，在 **Layout** 面板中，有一个 Flexbox 区域会展示页面中所有的 flexbox，可以快速的触发某个元素的 flex 布局浮层。

![](https://developer-chrome-com.imgix.net/image/BrQidfK9jaQyIHwdw91aVpkPiib2/pgEhguLqTSHn5AuiA7Ti.png?auto=format&w=1428)


## Core Web Vitals 浮层

Core Web Vitals 浮层可以更好的展示和测量页面性能。
[Core Web Vitals](https://web.dev/vitals/) 是 Google 提出的一项用于衡量页面性能指标的方案，这些性能指标对提升用户体验至关重要。打开 **Command Menu**， 执行 **Show Rendering**，然后在面板中选中 **Core Web Vitals** 选择框之后，浮层就会被打开。

浮层会展示以下三项数据：

- [Largest Contentful Paint (LCP)](https://web.dev/lcp/): 用于测量页面加载性能。为了良好的用户体验，页面第一次加载的时候 LCP 应该在 2.5 秒以内。
- [First Input Delay (FID)](https://web.dev/fid/): 用于测量页面可交互性。为了良好的用户体验，页面的时候 FID 应该在 100 毫秒以内。
- [Cumulative Layout Shift (CLS)](https://web.dev/cls/): 用于测量页面视觉稳定性。为了良好的用户体验，页面的 CLS 应该少于 0.1

 ![](https://developer-chrome-com.imgix.net/image/BrQidfK9jaQyIHwdw91aVpkPiib2/95Iw3l9ePIopJuApx65h.png?auto=format&w=1428)


## Issues tab 相关更新
### issue 的数量移到了 Console 状态栏

在 **Console 状态栏** 中加了一个新的按钮用于展示 issue 的数量，可以使 issue 的数量更明显的展现出来，在 **Console** 中不再展示 issue 信息。

![](https://developer-chrome-com.imgix.net/image/BrQidfK9jaQyIHwdw91aVpkPiib2/vg2AGCiq8IWkXU7MoHR9.png?auto=format&w=1428)

### 报告 Trusted Web Activity issues

当前，**Issues tab** 可以报告 [Trusted Web Activity](https://developer.chrome.com/docs/android/trusted-web-activity/overview/) issues，主要是为了帮助开发者理解和修复网站中的 Trusted Web Activity 问题，以提高应用的质量。

打开一个 Trusted Web Activity 应用，然后通过点击 **Console** 状态栏上的 issue 数量按钮来打开 **Issues** 标签，issue 的详细信息就会展示出来了。

![](https://developer-chrome-com.imgix.net/image/BrQidfK9jaQyIHwdw91aVpkPiib2/FSoAR540YOC6B86Cl7l7.png?auto=format&w=1428)

## 在 Console 中格式化字符串

**Console** 中可以将字符串格式化成合法的 JavaScript 字符串字面量。以前 **Console** 在输出字符串时，不会将双引号进行解码。

![](https://developer-chrome-com.imgix.net/image/BrQidfK9jaQyIHwdw91aVpkPiib2/4OPajz8MHz5lPMhPpzg5.png?auto=format&w=1428)

## Trust Tokens 面板

DevTools 在 **Application** 面板中加了一个叫 **Trust Tokens** 新面板，用于展示在当前浏览上下文中可用的 Trust Tokens。[Trust Token](https://web.dev/trust-tokens/) 是一种新的 API，用于帮助打击欺诈行为，并将机器人与真人区分开来，无需被动跟踪。

![](https://developer-chrome-com.imgix.net/image/BrQidfK9jaQyIHwdw91aVpkPiib2/j5idcrmLOWTIcd6vG0q9.png?auto=format&w=1428)

## 开启 CSS color-gamut 媒体特性 

![](https://developer-chrome-com.imgix.net/image/BrQidfK9jaQyIHwdw91aVpkPiib2/OpwDY2ebDj8aOzTR9RlS.png?auto=format&w=1428)

色域媒体查询（color-gamut media query）可用于测试浏览器和输出设备支持的大致颜色范围。例如，如果媒体查询与 `color-gamut: p3` 匹配，则表示用户的设备支持 `Display-p3` 颜色空间。

打开 **Command** 目录，执行 **Show Rendering** 命令，选择 **Emulate CSS media feature color-gamut** 下拉选项，就可以设置相关属性值了。

## Progressive Web Apps 工具新特性

DevTools 支持在 **Console** 中展示 [PWA](https://web.dev/progressive-web-apps/) 安装过程中更详细的警告信息。

![](https://developer-chrome-com.imgix.net/image/BrQidfK9jaQyIHwdw91aVpkPiib2/gotE1NFtBfITfdz4QDei.png?auto=format&w=1428)

如果 PWA manifest 的 `description` 超过 324 个字符，在 **Manifest** 面板里会展示警告信息。

![](https://developer-chrome-com.imgix.net/image/BrQidfK9jaQyIHwdw91aVpkPiib2/DIxxWCedyNCknMSvO1rb.png?auto=format&w=1428)


如果 PWA manifest 的 `screenshot` 不符合要求，在 **Manifest** 面板里会展示警告信息。

![](https://developer-chrome-com.imgix.net/image/BrQidfK9jaQyIHwdw91aVpkPiib2/lZzs7jdcln4N1D6PEBA6.png?auto=format&w=1428)


## Network 面板中添加了 `Remote Address Space`


每个网络请求的 IP 地址空间可以在 **Network** 中的 `Remote Address Space` 进行查看。

![](https://developer-chrome-com.imgix.net/image/BrQidfK9jaQyIHwdw91aVpkPiib2/Wz4Z5hiSK66IKoRWQGDj.png?auto=format&w=1428)


## 性能方面的提升

当 DevTools 打开时，提高了加载页面的性能。在某些极端情况下，性能可以提高 10 倍。

DevTools 收集堆栈跟踪，并将它们附加到控制台消息或异步任务中，以供开发人员在出现问题时使用。由于此收集必须在浏览器引擎中同步进行，因此在 DevTools 打开的情况下，缓慢的堆栈跟踪收集会显著降低页面的速度。我们已经设法显著降低了堆栈跟踪收集的开销。

## 在 **Frame** 详情视图中展示 allowed/disallowed 功能

在 **Frame** 详情视图中显示由 [Permissions policy](https://github.com/w3c/webappsec-permissions-policy/blob/main/permissions-policy-explainer.md) 控制的允许和不允许的功能列表。

Permissions policy 是一种 web 平台 API，它使网站能够允许或阻止在其自身的 frame 或其被嵌入的 iframe 中使用某些浏览功能。

![](https://developer-chrome-com.imgix.net/image/BrQidfK9jaQyIHwdw91aVpkPiib2/Ylt0ONDSndEn36MPmRUS.png?auto=format&w=1428)


## **Cookies** 面板中加 `SameParty` 列

应用程序面板中的 **Cookies** 面板增加了 `SameParty`， `SameParty` 属性是一个布尔值，用于说明同一 [First-Party Sets](https://github.com/privacycg/first-party-sets) 中的源是否可以在请求中包含 cookie。

![](https://developer-chrome-com.imgix.net/image/BrQidfK9jaQyIHwdw91aVpkPiib2/8qaTUkcNI1wv0qh4U00g.png?auto=format&w=1428)


## `fn.displayName` 被废弃

由于 `fn.displayName` 是非标准的属性，已经被废弃掉了，可以用 `fn.name` 来替代它。

![](https://developer-chrome-com.imgix.net/image/BrQidfK9jaQyIHwdw91aVpkPiib2/oXk5CGKAAPyJIQeecS0I.png?auto=format&w=1428)

Chrome 一直都支持非标准的 `fn.displayName`  属性，开发人员可将其做为函数的名称在 `error.stack` 和 DevTools 的堆栈中展示出来。以前，在上面的示例中，在调用堆栈中 `fn.displayName` 会显示为 `noLongerSupport`。

用来替换 `fn.displayName` 的是标准中的 `fn.name`，可以用 `Object.defineProperty` 来设置 `fn.name`。用 `fn.displayName` 在浏览器中显示名称是不可靠的，它减慢了堆栈跟踪收集的速度，无论开发人员是否真正使用它，开发人员总是要为此付出代价。

![](https://developer-chrome-com.imgix.net/image/BrQidfK9jaQyIHwdw91aVpkPiib2/aieojilTMlsewNdAaHHh.png?auto=format&w=1252)

## **Settings** 目录中的 `Don't show Chrome Data Saver warning`  被废弃

由于 [Chrome Data Saver](https://blog.chromium.org/2019/04/data-saver-is-now-lite-mode.html) 已经被废弃，所以 `Don't show Chrome Data Saver warning` 也被废弃了。

 ![](https://developer-chrome-com.imgix.net/image/BrQidfK9jaQyIHwdw91aVpkPiib2/d0T9rqp1ASln0kWXE41w.png?auto=format&w=1252)

 
## 实验特性

### Issues 标签页中加入 Automatic low-contrast issue 报告

开启方式: **Settings** > **Experiments** > 勾选 **Enable automatic contrast issue reporting via the Issues panel**

当页面中有 contrast issue 时，在 Issues 的标签页中会出现 issues。 出现 [contrast issue](https://web.dev/color-and-contrast-accessibility/) 意味着用户视觉上不易辨识页面的某些内容。

 ![](https://developer-chrome-com.imgix.net/image/BrQidfK9jaQyIHwdw91aVpkPiib2/rZE5UWUXF4K9JCMA3oSF.png?auto=format&w=1252)

### Elements 面板中支持完整的 accessibility tree 视图 

开启方式: **Settings** > **Experiments** > 勾选 **Full accessibility tree view in Elements pane**

在 **Elements** 面板中, 点击顶部的 `Switch to Accessibility Tree view` 切换到 accessibility tree 视图：

![](https://developer-chrome-com.imgix.net/image/BrQidfK9jaQyIHwdw91aVpkPiib2/pkTyHQF0YfbgiE9IZufX.png?auto=format&w=1252)