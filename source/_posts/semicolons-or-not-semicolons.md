---
title: JavaScript 要不要加分号
date: 2021-02-19 12:16:11
tags: JavaScript
---

正如粽子的甜咸之争，在前端也有分号党和不加分号党之间的争论。分号党认为前端代码必须要加分号，而不加分号党认为没有必要的情况可以不加分号。本文将对 JavaScript 的这个问题做一些简单讨论。

## 问题起源

JavaScript 并不要求在每个语句后面一定要加分号，因为在大部分情况下 JavaScript 引擎是可以识别出语句的结束位置并且自动插入分号，这个过程被称为 ASI ([Automatic Semicolon Insertion](https://262.ecma-international.org/6.0/#sec-automatic-semicolon-insertion) ) 。但是，并不是所有的语句都能省略分号，因为 ASI 解析规则会导致某些情况代码异常。

我们经常看到一些[类库是以分号开头](https://stackoverflow.com/questions/1873983/what-does-the-leading-semicolon-in-javascript-libraries-do)，譬如：

```js
;(function () {})()
```

如果没有前面的 `;` 当这段代码和别的代码合并到一起的时候，出现下面这种情况会报错：

```js
var name = 3
(function () {})()

// 由于没有分号，上面的会被解析为下面的语句，导致出现报错
var name = 3(function () {})()
```

如果你是无分号党，当遇见以 `+ - [ ( /` 为开头的语句的时候，要特别注意是否需要加分号，而且分号要加到行首。

在 ECMAScript 规范里没有规定必须加分号，而开发人员的有不同的开发习惯，以及 JavaScript 也无法做到全部不加分号，这些原因汇聚到一起，最终导致大家在这个问题上产生了分歧。

## 两者差别

《JavaScript 高级程序设计(第4版)》中有如下一段描述（具体见 22 页）：

> 即使语句末尾的分号不是必需的，也应该加上。记着加分号有助于防止省略造成的问题，比如可以避免输入内容不完整。此外，加分号也便于开发者通过删除空行来压缩代码(如果没有结尾的分号，只删除空行，则会导致语法错误)。加分号也有助于在某些情况下提升性能，因为解析器会尝试在合适的位置补上分号以纠正语法错误。

其中 `加分号有助于防止省略造成的问题`，是指全部代码加分号可以避免因缺少分号而导致错误。目前，C 端的JavaScript 代码几乎都需要经过 Webpack 等工具的压缩，压缩的结果可以选择是否保留分号。


## 代码规范

在不同的代码规范，对分号的限制也不一致。

[Google JavaScript Style Guide](https://google.github.io/styleguide/javascriptguide.xml) 代码规范中，需要加分号。

[Airbnb JavaScript Style Guide](https://github.com/airbnb/javascript) 代码规范中，必须要加分号:

```js
// https://github.com/airbnb/javascript#semicolons
// https://github.com/airbnb/javascript/blob/master/packages/eslint-config-airbnb-base/rules/style.js
semi: ['error', 'always']
```

[JavaScript Standard Style](https://github.com/standard/standard) 代码规范中，不允许使用分号:

```js
// https://github.com/standard/standard/blob/master/RULES.md#semicolons
// https://github.com/standard/eslint-config-standard/blob/master/eslintrc.json
"semi": ["error", "never"]
```
Airbnb JavaScript Style Guide 和 JavaScript Standard Style 对于分号的限制都是基于 [ESlint semi](https://eslint.org/docs/rules/semi.html)，但是比较有意思的是，两者的规则恰恰相反，并且两个规范都有大量的开发人员在使用。

## 大牛观点

知乎上有一篇问答 [JavaScript 语句后应该加分号么?](https://www.zhihu.com/question/20298345) 中，汇集了多位前端大牛的回答，大家可以做一些参考。

玉伯认为入乡随俗是一种态度：
> 看项目，如果是不加的项目，则不加，比如 zepto
> 如果是加的项目，则加上，比如 jquery
> 4 空格和 2 空格也是一样，两种风格我都习惯

尤雨溪则是坚定的不加分号党：

> 没有应该不应该，只有你自己喜欢不喜欢 ...
> Vue.js 的代码全部不带分号 ...
> 另外在我的强烈要求下 prettier 1.0 现在也支持无分号选项，在不同 style 之间迁移的成本已经接近 0 了 

## 最后总结

虽然个人的开发习惯和风格各不相同，但是需要在同一项目上保持一致，或者在团队内部达成一致。代码格式化工具和编辑器插件都能自动完成对分号的处理，也会很方便的做到代码风格的统一。

## 参考文档

- [https://262.ecma-international.org/6.0/#sec-automatic-semicolon-insertion](https://262.ecma-international.org/6.0/#sec-automatic-semicolon-insertion)
- [https://stackoverflow.com/questions/7145514/whats-the-purpose-of-starting-semi-colon-at-beginning-of-javascript](https://stackoverflow.com/questions/7145514/whats-the-purpose-of-starting-semi-colon-at-beginning-of-javascript)
- [https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Lexical_grammar#automatic_semicolon_insertion](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Lexical_grammar#automatic_semicolon_insertion)
- [https://eslint.org/docs/rules/semi](https://eslint.org/docs/rules/semi)
- [https://google.github.io/styleguide/javascriptguide.xml](https://google.github.io/styleguide/javascriptguide.xml)
- [https://github.com/standard/standard/blob/master/RULES.md#semicolons](https://github.com/standard/standard/blob/master/RULES.md#semicolons)
- [https://www.zhihu.com/question/20298345](https://www.zhihu.com/question/20298345)
- [https://github.com/twbs/bootstrap/issues/3057](https://github.com/twbs/bootstrap/issues/3057)
- [https://github.com/airbnb/javascript](https://github.com/airbnb/javascript)
- [http://slides.com/evanyou/semicolons#/](http://slides.com/evanyou/semicolons#/)
- [https://2ality.com/2012/09/expressions-vs-statements.html](https://2ality.com/2012/09/expressions-vs-statements.html)
- [https://www.zhihu.com/question/26673918](https://www.zhihu.com/question/26673918)