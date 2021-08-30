---
title: 彻底搞懂 JavaScript 数据类型转换
date: 2021-08-30 10:47:30
tags: 数据类型转换
---

在正文开始之前，先看这个代码：

```JavaScript
[] == ![] // ?
{} == ! {} // ?
```

![](https://p3.koolearn.com/843b9f8a.png)

这是啥？懵了...

隐约觉得是类型转换。

来来来，别懵，看完后面的文章，你就搞懂了。

## 强制类型转换

### Number()

**参数为原始类型**

```JavaScript
// 数值：转换后还是原来的值
Number(123) // 123

// 字符串：如果可以被解析为数值，则转换为相应的数值
Number('123') // 123

// 字符串：如果不可以被解析为数值，返回 NaN
Number('123abc') // NaN

// 空字符串转为0
Number('') // 0

// 布尔值：true 转成 1，false 转成 0
Number(true) // 1
Number(false) // 0

// undefined：转成 NaN
Number(undefined) // NaN

// null：转成0
Number(null) // 0
```

Number 函数将字符串转为数值，要比 parseInt 函数严格。基本上，只要有一个字符无法转成数值，整个字符串就会被转为 NaN。parseInt 可以处理字符串截取和数字进制问题：

```JavaScript
parseInt('32 abc') // 32
parseInt('0x11 abc') // 17

Number('32 abc') // NaN
Number('0x11 abc') // NaN
```

**参数为对象**

小规则：几乎都转为 NaN，除非是包含单个数值的数组。

```JavaScript
Number({ a : 1 }) // NaN
Number([ 1, 2, 3 ]) // NaN
Number([ 5 ]) // 5
```

为什么会这样？ 到底发生了什么？

先来看这样一段代码：

```JavaScript
const obj = {
  toString: function() {
    return 1;
  },
  
  valueOf: function() {
    return 2;
  }
};

console.log(Number(obj));  // 2
```

`Number(obj)` 调用 `valueOf` 方法，输出的结果为 `2`。将上面的代码做一些调整：

```JavaScript
const obj = {
  toString: function() {
    return 1;
  },
  
  valueOf: function() {
    return {};
  }
};

console.log(Number(obj)); // 1
```

结果为 `1`，`Number(obj)` 调用 `toString` 方法。那么再看下面的代码：

```JavaScript
const obj = {
  toString: function() {
    return {};
  },
  
  valueOf: function() {
    return {};
  }
};

console.log(Number(obj)); // 报错
```

我们会发现，执行代码的时候报错，错误信息 `Uncaught TypeError: Cannot convert object to primitive value`。如果删除 `toString` 方法呢？

```JavaScript
const obj = {
  valueOf: function(){
    return {};
  }
};

console.log(Number(obj)); // NaN
```

在上面的代码中， `Number({})` 的执行时调用了默认的 
`toString` 方法，也就是 `Object.prototype` 上的 `toString` 方法。具体的过程如下面的代码所示：

```JavaScript
Number({}) 
// 可以拆解为
Object.prototype.toString.call({}) //"[object Object]"
Number("[object Object]") // NaN
```

在上面的示例中，`valueOf` 返回的是对象，接下来我们看一下它返回数组的情形。

```JavaScript
const obj = {
  valueOf: function(){
    return [];
  }
};

console.log(Number(obj)) // 0
```

在上面的代码中 `Number(obj)` 可等价于 `Number([])`，这里调用的是 `Array` 的 `toString` 方法，而不是 `Object` 的 `toString`。如下：

```JavaScript
Number([])
// 可以拆解为
Array.prototype.toString.call([]) // ""
Number('') // 0
```

那么再看一下 `Object.prototype.toString`：

```JavaScript
Object.prototype.toString.call([]) //'[object Array]'
Number('[object Array]') // NaN 
```

可以得出下面的几点结论：

1. 调用对象自身的 `valueOf` 方法。如果返回原始类型的值，则直接对该值使用 `Number` 函数，不再进行后续步骤。

2. 如果 `valueOf` 方法返回的还是对象，则改为调用对象自身的 `toString` 方法。如果 `toString` 方法返回原始类型的值，则对该值使用 Number 函数，不再进行后续步骤。

3. 如果 `toString` 方法返回的是对象，报错。

Object.prototype.toString 返回值: 

```JavaScript
  // 原始类型
  Object.prototype.toString.call('123') // '[object String]'
  Object.prototype.toString.call(123) // '[object Number]'
  Object.prototype.toString.call(NaN) // '[object Number]'
  Object.prototype.toString.call(Infinity) // '[object Number]'
  Object.prototype.toString.call(null) // '[object Null]'
  Object.prototype.toString.call(undefined) // '[object Undefined]'
  
  // 引用类型
  Object.prototype.toString.call(function() {}) // '[object Function]'
  Object.prototype.toString.call([1,2,3]) // '[object Array]'
  Object.prototype.toString.call([1]) // '[object Array]'
  Object.prototype.toString.call({}) // '[object Object]'

  Object.prototype.toString.call(new Error()) // '[object Error]'
  Object.prototype.toString.call(/\d/) // '[object RegExp]'
  Object.prototype.toString.call(Date()) // '[object String]'
  Object.prototype.toString.call(new Date()) // '[object Date]'
  Object.prototype.toString.call(Symbol()) // '[object Symbol]'
  
  // 浏览器提供
  (function (){
    Object.prototype.toString.call(arguments) // '[object Symbol]'
  })()
  Object.prototype.toString.call(document) // '[object HTMLDocument]'
  Object.prototype.toString.call(window) // '[object Window]'
```

### String()

**参数为原始类型**

- 数值：转为相应的字符串

- 字符串：转换后还是原来的值

- 布尔值：true 转为字符串 'true'，false 转为字符串 'false'

- undefined：转为字符串 'undefined'

- null：转为字符串 'null'

```JavaScript
String(123) // "123"
String('abc') // "abc"
String(true) // "true"
String(undefined) // "undefined"
String(null) // "null"
```

**参数为对象**

```JavaScript
String({ a: 1 }) // "[object Object]"
String([1, 2, 3]) // "1,2,3"
```

```JavaScript
const obj = {
  toString: function() {
    return 1;
  },
  valueOf: function() {
    return 2;
  }
};
console.log(String(obj)) // 1
```

`String` 方法的转换规则，与 `Number` 方法基本相同，只是互换了`valueOf` 方法和 `toString` 方法的执行顺序。

1. 先调用对象自身的 `toString` 方法。如果返回原始类型的值，则对该值使用 `String` 函数，不再进行以下步骤。

2. 如果 `toString` 方法返回的是对象，再调用原对象的 `valueOf` 方法。如果 `valueOf` 方法返回原始类型的值，则对该值使用 `String` 函数，不再进行以下步骤。

3. 如果 `valueOf` 方法返回的是对象，报错。

不同类型的 `toString` 方法结果不同：

```JavaScript
[1, 2].toString() === Array.prototype.toString.call([1, 2]) // true

Array.prototype.toString.call([1, 2]) == Object.prototype.toString.call([1, 2]) // false
```

### Boolean()

**falsey (虚值)**

在 Boolean 类型转换时，虚值会转换为 `false`，虚值主要有以下几种：

```JavaScript
undefined
null
0 // +0 -0
NaN 
false 
'' 
```

**其他情况**

虚值之外的全部为 `true`：

```JavaScript
Boolean([]) // true
Boolean({}) // true
Boolean(new Error()) // true
Boolean(Symbol()) // true
```

## 隐式类型转换/自动转换

### 自动转换为布尔值

这些符号会自动转换为布尔值：

```JavaScript
if()
for while switch
! !! 
===
? :
&& ||
```

隐式类型转换: 除了虚值，其他都转换为 `true`

### 自动转换为字符串

字符串的自动转换，主要发生在字符串的**加法运算**时。当一个值为字符串，另一个值为非字符串，则后者转为字符串。

```JavaScript
'5' + 1 // '51'
'5' + true // "5true"
'5' + false // "5false"
'5' + {} // "5[object Object]"
'5' + [] // "5"
'5' + function (){} // "5function (){}"
'5' + undefined // "5undefined"
'5' + null // "5null"

null + [] ?
null + {} ?
null + funciton() {}
```

### 自动转换为数值

除了加法运算符（`+`）有可能把操作数转为字符串，其他算术运算符都会把操作数自动转成数值。

```JavaScript
'5' - '2' // 3
'5' * '2' // 10
true - 1  // 0
false - 1 // -1
'1' - 1   // 0
'5' * []    // 0
false / '5' // 0
'abc' - 1   // NaN
null + 1 // 1
undefined + 1 // NaN
```

注意：null 转为数值时为 0，而 undefined 转为数值时为 NaN。

正/负符号运算符中：

```JavaScript
+'abc' // NaN
-'abc' // NaN
+true // 1
-false // 0
```

关系运算符，如 `==` `!==`，也会转换为数值。

```JavaScript
1 == true // true
'1'== true // true
'1' == 1 // true
[1] == 1 // true
[1] == true // true
[] == false // true
[] == null // false
[1, 2] == NaN // false
```

### 特殊值

**Infinity**

```JavaScript
Number(Infinity) //Infinity
1 / 0 == Infinity // true
1 / Infinity == 0 // true
Infinity === Infinity // true
Infinity === -Infinity // false
0 === -0 // true
0 / 0 // NaN
```

**NaN**

```JavaScript
NaN == NaN // false
```

**undefined 与 null**

```JavaScript
null === null // true
undefined === undefined // true

undefined == null // true
undefined === null // false
```

## 总结

现在我们解答一下开始提到的问题。

首先是 `[] == ![]` 为 `true`，如下所示：

```JavaScript
// 拆解 [] == ![]

[].toSring() // ''
Number('')   // 0

![] // false
Number(false) // 0

0 == 0  // [] == ![] 为 true
```

而 `{} == !{}` 为 `false`，如下所示：

```JavaScript
// 拆解 {} == !{}

({}).toString() // "[object Object]";
Number("[object Object]") // NaN

!{} // false
Number(false) // 0

NaN != 0 // {} == !{} 为 false
```

那么，下面代码会输出什么呢？

```JavaScript
{} + {} // ?
[] + {} // ?
{} + [] // ?
```

可以点击这个[页面](https://jsisweird.com/)试一下。
