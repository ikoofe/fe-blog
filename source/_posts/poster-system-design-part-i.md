---
title: 前端实现高效的海报系统
date: 2021-07-20 15:18:11
tags: JavaScript

---

## 前言

众所周知，海报作为一种最为常见的招贴形式，被广泛应用于广告宣传等场景。随着互联网的快速发展，网络营销逐渐成为引流和获客的重要手段，形式多样的海报也在线上营销裂变活动中发挥着不可或缺的作用。与传统的招贴海报相比，线上海报则更加灵活多变，通常会为不同的用户生产制作出差异化的海报，以满足业务上的个性化诉求。因此，如何稳定高效地生产高质量的海报变得尤为重要。接下来我们将通过一系列文章，来介绍前端如何打造一个高效的海报系统。

为了方便理解，首先要明确的是，本文提到的海报是指海报图片，通常会带有二维码或菊花码，引导用户访问活动、加微信号、关注公众号等。举个例子，下面的这张海报就是在海报系统中制作的。**感兴趣的同学可以扫码关注公众号或扫码进入微信群交流。**

![](https://pic3.koocdn.com/7f39986862a502a16faf17978031e57cbe7c712a.png)

## 业务背景

在用户增长的业务场景中，以拼团、转介绍等为代表的各类裂变活动中都会使用海报作为载体，通过社群或朋友圈等渠道引起用户的关注，最终吸引用户参与到这些活动中来。为了提升和评估活动的投放效果，海报会包含用户的一些个性化的文案和信息。通常，这些海报在活动中是动态生成的，设计师设计出来的海报并不能直接在用户侧展示。显然，设计师也不可能为每个用户制作一张海报，所以需要有一些工具或系统来高效的生产海报。基于这样的业务背景，我们自研了一套海报系统，打通了海报的制作和使用中的各个环节，提高了海报的生产效率，降低了海报的开发成本。

海报系统接入了公司的设计素材库，提供了海量的素材，方便了设计师、产品、运营等相关人员制作海报。海报系统支持在线编辑，并且能将设计稿一键解析为海报，降低了海报系统的接入成本。另外，海报系统是在服务端生成海报，海报的清晰度以及稳定性方面也有了可靠的保障。任何系统都不是一蹴而就的，都会有一个演进和迭代的过程，海报系统也不例外。接下来将重点介绍海报系统的设计实现过程。

## 技术方案

海报本质上就是一张图片，在各类活动中海报会被转发或分享而传播出去，当其他用户看到海报时会关注到该活动。那么如何将海报图片生产出来并显示给用户呢？海报系统的方案是，将海报的生产和制作 `模版化`，将海报的绘制和渲染 `引擎化`。通过对海报进行分析，可以将海报的内容元素抽象成两类：

  - 基础元素：最基本的海报组成元素，包括 `线条`、`几何图形`、`图片`、`文本` 等
  - 业务元素：具有业务属性的元素，例如 `二维码`、`小程序码`、`头像` 等

模版化是将上述的元素做为最小单位，用可配置化的数据结构来描述海报，这些配置数据被称为海报模版。通过海报系统的模版编辑器，可以生成海报模版。

引擎化是提供统一海报渲染能力，解析海报模版并接收个性化参数，最终绘制出海报图片展示给用户。通过海报系统的渲染引擎，可以将海报模版绘制成海报图片。

![](https://p3.koolearn.com/05d9f552.jpeg)

### 海报模版

模版化的基于思路是定义了一套描述海报模版的 DSL，而渲染引擎能够将 DSL 解析并渲染出来。在实现中是以 JSON 作为 DSL 来描述海报模版。海报系统最核心的功能是将拖拽生成的海报模版存储为 JSON 数据，然后在运营活动中将这些 JSON 数据重新渲染为海报呈现给用户。 生成的 JSON 数据示例如下：

```JSON
{
  "width": 1920,
  "height": 720,
  "textArr": [
    {
      "id": 3,
      "drawType": "text",
      "text": "《前端实现高效的海报系统》",
      "x": 100,
      "y": 102.6,
      "zIndex": 3,
      // ...
    }
    // ...
  ]
}
```
### 海报渲染

在海报系统的最初实现中，渲染海报图片是在客户端实现的。但是随着系统的不断迭代，发现客户端渲染存在一些弊端，而且这些问题无法解决。最终，改造为服务端渲染海报，下面还是分别介绍两种渲染的实现方式。

**客户端渲染海报**

客户端渲染海报，顾名思义就是在用户的设备上（例如 页面、客户端等）渲染海报。我们调研了目前行业内比较主流的海报渲染解决方案，大致总结为以下两类：

1. 使用 HTML 元素制作和渲染海报，由 `html2cavnas` 转换成图片
2. 使用 Canvas 开发绘制海报，由 Canvas 生成图片


这两种方案的共同点都是在用户端生成海报，我们对这两种方案在开发效率、上手成本、生成性能及海报清晰度方面做了对比：

方案 | `html2cavnas` | Canvas
---|---|---
开发效率 | 高 | 低
上手成本 | 低 | 高
生成性能 | 低 | 高
海报清晰度 | 低 | 高

我们对 `html2cavnas` 方案进行了快速的验证，该方案在实际应用中存在以下致命的问题：

- **质量较差** 在某些情况下，由于生成的海报过于模糊，导致海报上的二维码无法被手机识别出来
- **性能较差** 在低端的安卓手机上，会因为兼容性等问题导致海报生成失败

最初，海报系统中渲染绘制海报图片的过程是在客户端实现的。渲染引擎将海报在 Canvas 中绘制出来，然后转化成图片提供给用户下载或分享。在客户端中受客户端机器设备、网络环境等各种因素的影响，海报渲染引擎存在着性能瓶颈。尽管对渲染引擎进行了几轮优化，在弱网环境、低性能安卓机上生成海报的耗时依然较长，存在的问题很难解决。基于用户网络环境、设备性能、客户端网络并发限制等多方面综合考虑，最终决定把海报生成解决方案进行服务化，将海报的配置、渲染全部迁移到服务端来完成。于是将客户端渲染改为服务端渲染海报。

**服务端渲染海报**

通过调研发现，在服务端渲染海报的实现方式主要有两种：

1. 用 `Puppeteer` 或 `Phantomjs` 截图生成图片
2. 用 `node-cavnas` 绘制并导出图片

 Puppeteer 或 Phantomjs 的实现方案与 html2canvas 比较相似，都需要把海报在网页中渲染出来；只是生成图片的方式存在差异，前者是对网页进行截图，后者是将网页转换成 Canvas 后再导出图片。最终，我们结合客户端开发渲染引擎的思路和方案采用了 node-canvas 进行服务端渲染引擎的设计与开发。在海报编辑器中海报是在 Canvas 中渲染完成的，服务端使用 node-canvas 可以统一出一套渲染引擎 SDK。这样既可支持 Web 页面上渲染海报，也支持 Node 服务上渲染海报。

 虽然，为 C 端用户生成的海报图片是服务端渲染出来的，在模版编辑器中海报的实时渲染依然用的是 Web 端渲染。因而，客户端渲染也没有完全被废弃掉。

 ![](https://p3.koolearn.com/e0ee20be.jpeg)

## 系统实现

整个海报系统分为海报管理后台和海报渲染服务两个部分：

- **海报管理后台** 设计、运营或产品可在海报管理后台中制作海报或模版。用户在可视化编辑器中，可以通过拖拽的方式编辑海报，也可以通过设计稿导入的方式制作海报。

- **海报渲染服务** 该服务将海报模版渲染成海报，把渲染成的海报上传到 CDN；业务系统通过 API 接口获取海报图片的 CDN 地址，并将海报图片展示给用户。

![](https://p3.koolearn.com/c9909b97.jpeg)

### 海报管理后台

海报管理后台的主要功能模块包括模版管理、素材管理、海报编辑等组成，如下图所示：

![](https://p3.koolearn.com/f7f02297.jpeg)

在这些功能模块中，最为复杂的模块是海报编辑。海报编辑支持静态海报编辑和动态海报模版编辑。海报管理后台的编辑区域如下图所示：

![](https://p3.koolearn.com/1e795b3d.jpg)

### 海报渲染服务

**服务验签** 海报系统采用了服务注册机制，接入方在接入海报系统时需要在系统进行注册获得相应的服务 ID 和密钥，在使用系统生成海报时需要使用服务 ID 和密钥完成服务校验后方可进行海报渲染。

**海报渲染** 在渲染海报之前，首先检查缓存中是否已经存在相同渲染数据的海报，如果存在则直接返回海报的 CDN 链接，如果不存在要做模版解析。将模版中的元素解析并渲染到 Canvas 画布中，最终生成 PNG 格式的图片上传到 CDN，并存储 CDN 链接。

**存储控制** 为了提升性能，海报渲染服务将已渲染的海报进行缓存，同一个用户在一个时间端内多次访问某个活动的海报时，都会从 Redis 缓存中获取海报链接，而不需要重新渲染海报。另外，海报图片的链接地址也会持久化存储到 MYSQL 数据库中。海报的渲染耗费较多的服务器资源，通过对海报链接进行存储处理，可以提高海报渲染服务的吞吐量。

## 上线效果

海报系统已经被广泛应用到增长、双师、销售等业务中，可以快速为业务系统生产海报。海报系统降低了制作海报的门槛，让产品、运营等非设计人员也有了制作海报的能力。
在海报系统中，非研发人员可以使用模版库和素材库自主编辑生成静态海报直接导出，同时也可以基于海报模版进行批量生成海报；研发人员可开发配置生成自定义的海报模版，可使用服务接口动态生成海报。

![](https://p3.koolearn.com/43da80bd.gif)

随着海报系统被应用的范围越来越广，对海报渲染的性能提出了更高的要求。服务端渲染很好的保障了海报渲染的速度和质量，通常渲染一张简单的海报只需要 `90ms`，而复杂海报的渲染时间是在 `300ms` 左右。

## 总结

在新东方在线的多个业务场景中，海报系统都发挥了不可替代的作用。本文主要介绍了海报系统的业务背景、技术方案和系统实现等内容。大量的技术细节（例如 渲染性能优化、DSL 模版解析、编辑器实现 等），由于篇幅原因并未展开讨论，在后续的文章中会做进一步介绍。关注文中海报的二维码，可以及时关注后续更新。
