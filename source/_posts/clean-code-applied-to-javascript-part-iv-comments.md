---
title: JavaScript 代码整洁之道-注释篇
date: 2021-04-29 14:35:11
tags: JavaScript
---

> 本文源于翻译 [Clean Code Applied to JavaScript — Part IV. Comments](https://dev.to/carlillo/clean-code-applied-to-javascript-part-iv-comments-4a7a)

# JavaScript 代码整洁之道-注释篇

在这篇文章中，我们将要讨论开发人员书写整洁代码时很有争议的一个主题：**注释**。

部分人认为，对代码进行注释是一个好的实践，但也有一部分人完全相反，认为注释不是好的做法。

首先，在代码是否需要注释上，没有绝对的规则，一切视情况而定。事实上，在许多情况下，注释对软件开发没有帮助，因为现在已经有很多比注释更好的工具。在另外一些情况下，注释可能会对我们现在的代码开发或将来的代码阅读造成干扰。因此，在这几种情况下，没有注释才是最佳实践。

另一方面，添加注释可能是良好的实践，例如虽然公开的 API 文档可用于了解库的对外使用方式，但是它不能解释其内部代码逻辑。

接下来，将总结几种常见的注释方式，错误的注释会对代码阅读产生一定的干扰。在实践中要避免错误的注释方式，从而提高代码的质量。

## 只注释业务逻辑复杂的内容

注释的目的往往是帮助程序员去更好的理解程序或者业务逻辑，因为有些逻辑对于程序员来说是很复杂的，直接阅读代码并不是很好理解。但往往代码的注释并不会去描述算法和逻辑，通常只是解释这部分代码的作用。程序真正的逻辑还是需要阅读源代码来理解，但同时，优秀的代码本身就是注释，通常都能所见即明白。因此，给代码添加注释可以有，但通常不是必需的。

也有一些例外情况，比如程序中存在一个特定的业务逻辑，我们作为开发人员不知道该逻辑。例如，信用卡的业务逻辑，保险的业务逻辑等。不对其进行解释，阅读起来会很难明白这段代码的目的。下面的示例显示了大多数代码操作中的细节信息和不必要的注释。但最后一条注释是有用的，因为它不是在描述编程级别的操作，而是对业务逻辑的解释。

```JavaScript
function convert(data){
 // The result
 let result = 0;

 // length of string
 const length = data.length;

 // Loop through every character in data
 for (let i = 0; i < lenght; i++){
   // Get character code.
   const char = data.charCodeAt(i);
   // Make the hash
   result = (result << 5) - result + char;
   // Conver to 32-bit integer
   result &= result;
  }
}
```

在上面的代码中，大部分注释并没有什么作用，反而对我们的阅读代码产生了干扰，等效的代码如下所示。

```JavaScript
function convert(data) {
  let result = 0;
  const length = data.length;

  for (let i = 0; i < length; i++){
    const char = data.charCodeAt(i);
    result = (result << 5) - result + char;
    result &= result; // Convert to 32-bit integer
  }
}
```

## 避免日志型注释

以日期为维度的日志型注释，能随着时间的推移，记录了文件内容的变化。在过去缺少版本控制工具的时代，这种注释是有意义的。但是现在已经不再推荐这种做法。这些记录日志的工作应该交给版本控制工具(比如 git)，因此，不需要无效的代码，比如这种注释，会增加代码的冗余程度，不够整洁。如果需要查看此类的信息，可以使用 `git log` 来获取历史记录。

下面是一个带有日志型注释的代码，和无注释的干净版本。

```JavaScript
/**
 * 2018-12-20: Removed monads, didn't understand them (CC)
 * 2018-10-01: Improved using special mondas (JS)
 * 2018-02-03: Removed type-checking (LI)
 * 2017-03-14: Added add with type-checking (CC)
 */
 function add(a, b) {
   return a + b;
 }
```
```JavaScript
function add(a, b) {
   return a + b;
 }
```

## 避免使用注释去标记位置

应该避免使用注释进行位置标记，因为这种做法通常只会让代码更冗余。正确的对变量和函数命名，优化代码组织结构，增加代码的可读性。

下面的代码是一个带有位置标记注释的示例及其无注释版本。不难发现，使用这种注释已经过时了，现在已经没有必要在源代码中创建这些标记。

```JavaScript
///////////////////////////////
//  Controller Model Instantiation
///////////////////////////////
controller.model = {
  name: 'Felipe',
  age: 34
};

///////////////////////////////
//  Action Setup
///////////////////////////////
const actions = function() {
  // ...
};
```
```JavaScript
controller.model = {
  name: 'Felipe',
  age: 34
};

const actions = function() {
  // ...
};
```
## 总结

是否需要添加注释是当今软件开发者争议最大的主题之一，有些开发者认为注释是必要的，而有些则认为注释不是必要的。在人生的任何选择中，极端永远不是最好的选择，软件开发亦是如此。

所以，在这篇文章中总结了注释代码的三种做法。但是，如果我们正在创建一个公共 API，那么编写注释可能会很有趣，因为这些注释可以看作是 API 的说明文档。

一个非常坏的习惯是在代码的每一行都添加注释。很多初级程序员在学习写代码时候养成了这种坏习惯，把注释每一行代码当作在做学习笔记。

在软件开发过程中，做学习笔记和添加代码注释之间的差异是巨大的，不应该混淆。

最后，我们再回顾一下本文的重点：
- 只注释业务逻辑复杂的内容
- 避免日志型注释
- 避免使用注释去标记位置