---
title: JavaScript 中几种循环遍历方式对比 
date: 2021-01-30 11:08:02
tags:
---

> 本文来源于翻译文章 [Which type of loop is fastest in JavaScript?](https://medium.com/javascript-in-plain-english/which-type-of-loop-is-fastest-in-javascript-ec834a0f21b9)，文章内容主要是对 JavaScript 中的四种常用的循环语句的性能做了一些对比，并且各个应用场景做了简单概括。原文的内容比较水干货较少，但是提到一个比较有用的观点：要重视代码的可读性。

JavaScript 在 Web 开发上占有举足轻重的地位，无论是 NodeJS、React、Angular、Vue 等各种 JavaScript 框架， 还是 vanilla JS（译者注：这是个梗，指原生JavaScript） 都有大量的粉丝簇拥。循环作为编程语言中不可或缺的一部分，JavaScript 提供了多种支持循环遍历的方法，比如 `for`、 `for(reverse)`、 `for...of`、 `foreach`、  `for...in`、  `for...await`。但是问题是，如何在这些的循环遍历方法中，选择最适合我们需求的那个呢？这就是本文将要展开讨论的问题。

## 哪种循环遍历方法最快?

答案是：`for(reverse)`

在个人电脑上，对这几种遍历方式进行测试，最终得出了 `for(reverse)` 是最快的循环遍历。测试的方法是，遍历一个有 100 万个元素的数组，计算出整个过程的耗时，代码如下：

```js
const million = 1000000; 
const arr = Array(million);
console.time('⏳');

for (let i = arr.length; i > 0; i--) {} // for(reverse) :- 1.5ms
for (let i = 0; i < arr.length; i++) {} // for          :- 1.6ms

arr.forEach(v => v)                     // foreach      :- 2.1ms
for (const v of arr) {}                 // for...of     :- 11.7ms

console.timeEnd('⏳');
```

这里 `for` 的正向遍历和反向遍历耗时几乎是一样的，只有 0.1ms 的差异。原因是 `for(reverse)` 只进行一次 `let i = arr.length` , 而在 `for` 中每次都要进行 `i < arr.length` 判断，针对这点细微的差别可以忽略。和 `for` 相比，`foreach` 和 `for...of` 在数组遍历过程会更耗时。

> 译者注：如果将 `for (let i = 0; i < arr.length; i++) {}` 改为 `for (let i = 0, length = arr.length; i < length; i++) {}` 那么是不是就没有什么差异了呢？


## 不同循环遍历方法的应用场景

### for 循环

`for` 是大家较为熟悉的循环遍历方式，而且它的遍历速度是最快的，那是不是什么场景都推荐用 `for` 呢？答案是否定的，因为除了要考虑性能因素之外，代码的可读性通常更为重要。

### forEach

```js
const things = ['have', 'fun', 'coding'];
const callbackFun = (item, idex) => {
    console.log(`${item} - ${index}`);
}
things.foreach(callbackFun); 

o/p:- have - 0
      fun - 1
      coding - 2
```

`forEach` 在数组遍历过程中，不能被 `break` 或 `return` 提前结束循环。

### for ... of

`for...of` 是 ES6 支持的特性，用于遍历可迭代的对象，例如 `String`、`Array`、`Map` 和 `Set` 等，它对于代码可读性比较好。

```js
const arr = [3, 5, 7];
const str = 'hello';
for (let i of arr) {
   console.log(i); // logs 3, 5, 7
}
for (let i of str) {
   console.log(i); // logs 'h', 'e', 'l', 'l', 'o'
}
```

> 译者注：Airbnb 的代码规范中是不推荐使用 `for...of` 语句的。如果在 eslint 配置了 `eslint-config-airbnb`，当代码中使用了 `for...of` 会提示 `iterators/generators require regenerator-runtime, which is too heavyweight for this guide to allow them. Separately, loops should be avoided in favor of array iterations.` 虽然 Airbnb 的代码规范维护者，同样认为代码的可读性要比性能更重要，但是 `for...of` 的迭代遍历的底层是依赖于 `Symbol`，需要引入 [regenerator-runtime](https://www.npmjs.com/package/regenerator-runtime) 来做支持。为了用 `for...of` 而引入一个额外的库，付出的成本有点高

### for...in

`for...in` 可以遍历访问对象的所有可枚举属性。当用 `for...in` 访问数组的时候，除了返回数组的索引之外，数组上的用户自定义属性也会被返回，所以要避免用 `for...in` 遍历数组。

> 译者注：为了解释这个问题，可以参考下面这段代码

```js
const array = ['k', 'o', 'o'];

array.koo = true;

for (const key in array) {
  console.log(key); // 顺次打印 '0', '1', '2', 'koo'
}
```

> 译者注：Airbnb 的代码规范中也不推荐使用 `for...in` 来遍历对象的属性，推荐的方法是使用 `Object.{keys,values,entries}`


## 总结
- `for` 速度最快, 但可读性差
- `foreach` 速度快, 可控制属性
- `for...of` 比较慢, 但好用
- `for...in` 比较慢, 最不好用

最后给一个建议，把代码的可读性放在第一位。当开发一个复杂的结构（系统）时，代码可读性是必不可少的，但是也应该关注性能。尽量避免在代码中添加不必要的东西，以减少对应用程序性能造成的影响。

---

参考阅读

- [Using 'ForOfStatement' is not allowed](https://github.com/airbnb/javascript/issues/1271)

- [Array iteration methods summarized](https://gist.github.com/ljharb/58faf1cfcb4e6808f74aae4ef7944cff)

- [Airbnb JavaScript Style Guide](https://github.com/airbnb/javascript)