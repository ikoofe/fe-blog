---
title: 中文技术文章排版
date: 2021-06-03 16:59:49
tags: 
---

# 中文技术文章排版

## 标题

### 标题层级

标题分为四级。

- 一级标题：文章的标题
- 二级标题：文章主要部分的大标题
- 三级标题：二级标题下面一级的小标题
- 四级标题：三级标题下面某一方面的小标题

标题示例：

```Markdown
# 一级标题
## 二级标题
### 三级标题
#### 四级标题
```

### 标题原则

**1. 各级标题不能出现跨级**

错误示例：

```Markdown
# 一级标题

### 三级标题
```

**2. 标题避免重名**

错误示例：

```Markdown
## 错误示例

### 错误示例
```

## 空格

### 中英文之间需要增加空格

正确示例：

```Markdown
本文中所有示例都是用 JavaScript 来实现
```

错误示例：

```Markdown
本文中所有示例都是用JavaScript来实现。
```

### 中文与数字之间需要增加空格

正确示例：

```Markdown
理想情况下入参个数应该不多于 2 个。
```

错误示例：

```Markdown
理想情况下入参个数应该不多于2个。
```

## 标点符号

### 中文语句中的标点符号使用全角符号

正确示例：

```Markdown
接下来，我们开始介绍如何将这些技巧应用在具体的开发实践中。
```

错误示例：

```Markdown
接下来, 我们开始介绍如何将这些技巧应用在具体的开发实践中.
```

### 括号内的内容为纯英文和数字或半角英文符号时用半角括号，括号前后加空格

正确示例：

```Markdown
这些良好的实践适用于任何编程语言，包括最接近硬件 (CTM) 的编程语言。
```

错误示例：

```Markdown
这些良好的实践适用于任何编程语言，包括最接近硬件（CTM）的编程语言。
```

## 专业术语要正确

英文的专业术语要书写正确，特别注意字母的大小写以及不能随意省略。相关用词可以参照 [MDN 术语表](https://developer.mozilla.org/en-US/docs/Glossary)。

正确示例：

```Markdown
学习 JavaScript 的正确姿势。
```

错误示例：

```Markdown
学习 Javascript 的正确姿势。

学习 Js 的正确姿势。
```

## 代码

### 明确代码所用的语言

Markdown 中的代码块，明确语言类型之后，代码才能正确高亮显示。

正确示例：

```JavaScript
const name = 'koo';
console.log(name)
```

错误示例：

```
const name = 'koo';
console.log(name)
```

### 代码格式要优雅

空格、Tab、分号等用法要统一。

正确示例：

```JavaScript
const name = 'koo';
if (name === 'koo') {
  console.log('Yes');
} else {
  console.log('No');
}
```

错误示例：

```JavaScript
let name = 'koo';
if (name === 'koo') {
  console.log('Yes')
} else {
    console.log('No');
}
```

---

**参考文档**
- [中文博客排版指南](https://github.com/qianguyihao/document-guide)
- [中文技术文档的写作规范](https://github.com/ruanyf/document-style-guide)
- [中文排版指南](https://github.com/ctf-wiki/ctf-wiki/wiki/%E4%B8%AD%E6%96%87%E6%8E%92%E7%89%88%E6%8C%87%E5%8D%97)
- [译文排版规则指北](https://github.com/xitu/gold-miner/wiki/%E8%AF%91%E6%96%87%E6%8E%92%E7%89%88%E8%A7%84%E5%88%99%E6%8C%87%E5%8C%97)

