---
title: ES2021 带来的新特性
date: 2021-01-24 20:23:51
tags: ES2021
---


# ES2021 带来的新特性

ES2021（ES12）将会带来 5 个新特性，本文将对这些新特性做一些简单的介绍。

## String.prototype.replaceAll()

在 JavaScript 中，通常会使用 `String.prototype.replace()` 方法来完成字符串的替换，例如：

```js
'koofe'.replace('o', 'ô');
// kôofe
```

在 `String.prototype.replace()` 方法中，当第一个参数是字符串类型时，只替换第一个匹配的字符串，相关细节可以参阅 [MDN 文档](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/replace)。因此，在上面的代码中，只有第一个 `o` 被替换成了 `ô`，第二个 `o` 保持不变。如果要将匹配的字符串全部替换，只能通过正则表达式实现，例如：

```js
'koofe'.replace(/o/g, 'ô');
// kôôfe
```

为了方便字符串的全局替换，ES2021 将支持 `String.prototype.replaceAll()` 方法，可以不用写正则表达式就可以完成字符串的全局替换，例如：

```js
'koofe'.replaceAll('o', 'ô');
// kôôfe
```

更多阅读
- [https://stackoverflow.com/questions/1144783/how-to-replace-all-occurrences-of-a-string](https://stackoverflow.com/questions/1144783/how-to-replace-all-occurrences-of-a-string)
- [https://github.com/tc39/proposal-string-replaceall](https://github.com/tc39/proposal-string-replaceall)
- [https://v8.dev/features/string-replaceall](https://v8.dev/features/string-replaceall)
- [https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/replaceall](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/replaceall)
- [https://caniuse.com/?search=replaceAll](https://caniuse.com/?search=replaceAll)

## Promise.any

`Promise` 支持以下几种方法：

- `Promise.allSettled`: 无论每一个 promise 是fulfilled 还是 rejected，返回值都是一个 resolved promise

  ```js
  Promise.allSettled([
    Promise.reject(1),
    Promise.resolve(2)
  ])
  .then(result => console.log('result:', result))
  .catch(error => console.error('error:', error));

  // result:
  // [{ status: "rejected", reason: 1 },
  // { status: "fulfilled", value: 2 }]
  ```


- `Promise.all`: 只要有一个 promise 是 rejected，则立即返回一个 rejected promise；所有的 promise 都是 fulfilled 时，则返回一个 resolved promise

  ```js
  Promise.all([
    Promise.reject(1),
    Promise.resolve(2)
  ])
  .then(result => console.log('result:', result))
  .catch(error => console.error('error:', error));

  // error: 1
  ```

- `Promise.race`: 只要有 promise 是 fulfilled 或 rejected，则立即返回一个 resolved 或 rejected promise

  ```js
  Promise.race([
    Promise.reject(1),
    Promise.resolve(2)
  ])
  .then(result => console.log('result:', result))
  .catch(error => console.error('error:', error));

  // error: 1
  ```

- `Promise.any`: 只要有一个 promise 是 fulfilled 时，则返回一个 resolved promise；所有的 promise 都是 rejected 时，则返回一个 rejected promise

  ```js
  Promise.any([
    Promise.reject(1),
    Promise.resolve(2)
  ])
  .then(result => console.log('result:', result))
  .catch(error => console.error('error:', error));

  // result: 2
  ```

以上四个方法中，`Promise.all` 和 `Promise.race` 是 ES2015 中的特性，而 `Promise.allSettled` 和 `Promise.any` 分别是 ES2020 和 ES2021 中的特性。


更多阅读
- [https://github.com/tc39/proposal-promise-any](https://github.com/tc39/proposal-promise-any)
- [https://v8.dev/features/promise-combinators](https://v8.dev/features/promise-combinators)


## 逻辑赋值运算符

逻辑赋值运算符由逻辑运算符和赋值表达式组合而成：
- or or equals (||=)
  
  ```js
  a ||= b;

  // 与 a ||= b 等价
  a || (a = b);

  // 与 a ||= b 等价
  if (!a) {
    a = b;
  }
  ```
- and and equals (&&=)

  ```js
  a &&= b;

  // 与 a &&= b 等价
  a && (a = b);

  // 与 a &&= b 等价
  if (a) {
    a = b;
  }
  ```

- question question equals (??=)
  
  ```js
  a ??= b;

  // 与 a ??= b 等价
  a ?? (a = b);

  // 与 a ??= b 等价
  if (a === null || a === undefined) {
    a = b;
  }
  ```

用法示例：

```js
let obj = {};

obj.x ??= 0;
console.log(obj.x) // 0

obj.x ||= 1;
console.log(obj.x) // 1

obj.x &&= 2;
console.log(obj.x) // 2
```

注意事项：
```js
a = a || b; // 与 a ||= b 不等价
a = a && b; // 与 a &&= b 不等价
a = a ?? b; // 与 a ??= b 不等价
```

不等价的原因在于，无论上面的每个逻辑表达式结果如何，都会进行赋值运算；而逻辑赋值运算符只会在条件成立的情况下进行赋值，比如：

```js
let x = 0;
const obj = {
  get x() {
    return x;
  },
  
  set x(value) {
    console.log('setter called');
    x = value;
  }
};

// This always logs "setter called"
obj.x += 1;
assert.equal(obj.x, 1);

// Logical operators do not call setters unnecessarily
// This will not log.
obj.x ||= 2;
assert.equal(obj.x, 1);

// But setters are called if the operator does not short circuit
// "setter called"
obj.x &&= 3;
assert.equal(obj.x, 3);
```

更多阅读
- [https://github.com/tc39/proposal-logical-assignment](https://github.com/tc39/proposal-logical-assignment)


## 数字分隔符
使用 `_` 对数字进行分割，提高数字的可读性，例如在日常生活中数字通常是每三位数字之间会用 `,` 分割，以方便人快速识别数字。在代码中，也需要程序员较便捷的对数字进行辨识：

```js
// 1000000000 不易辨识
const count1 = 1000000000;

// 1_000_000_000 很直观
const count2 = 1_000_000_000;

console.log(count1 === count2); // true
```

## WeakRefs

`WeakRef` 实例可以作为对象的弱引用，对象的弱引用是指当该对象应该被 GC 回收时不会阻止 GC 的回收行为。而与此相反的，一个普通的引用（默认是强引用）会将与之对应的对象保存在内存中。只有当该对象没有任何的强引用时，JavaScript 引擎 GC 才会销毁该对象并且回收该对象所占的内存空间。因此，访问弱引用指向的对象时，很有可能会出现该对象已经被回收，`WeakRef` 使用方法如下：

```js
const ref = new WeakRef({ name: 'koofe' });
let obj = ref.deref();
if (obj) {
  console.log(obj.name); // koofe
}
```

> 对于 WeakRef 对象的使用要慎重考虑，能不使用就尽量不要使用

更多阅读
- [https://github.com/tc39/proposal-weakrefs](https://github.com/tc39/proposal-weakrefs)
- [https://github.com/tc39/proposal-weakrefs/blob/master/reference.md](https://github.com/tc39/proposal-weakrefs/blob/master/reference.md)
- [https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/WeakRef](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/WeakRef)

## 总结

目前，ES2021 正处于 Stage 4 阶段，也就是可以被纳入到正式的 ECMAScript 语言规范中了，预计今年年中会被正式发布。本文介绍的这几个新特性，在最新版本的 Chrome 浏览器中均已支持，感兴趣的话可以快速体验一番。