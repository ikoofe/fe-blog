---
title: 彻底搞懂，JS数据类型转换
date: 2021-08-30 10:47:30
tags: 据类型转换
---
## 开始之前，先看这个代码

```JavaScript
[] == ![]
{} == ! {}
```

![](https://p3.koolearn.com/f58ff8d0.jpeg)

这是啥？ 懵了...

隐约觉的是考察类型转换。

来来来，别懵，看完后面的文章，你就搞懂了。

## 1.强制类型转换

### 1.1 Number()

#### （1） 参数为原始类型

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

Number函数将字符串转为数值，要比parseInt函数严格。基本上，只要有一个字符无法转成数值，整个字符串就会被转为NaN。

```JavaScript
parseInt('32 abc') // 32
Number('32 abc') // NaN
```

```JavaScript
Number() 
parseInt() 
// 将字符串转为整数
parseFloat() 
// 将字符串转为浮点数
parseInt() 
//将字符串转为整数 弥补Number不足
// 1.字符串截取
// 2.进制问题 2-36

Number('0x11 abc') // NaN
parseInt('0x11 abc') // 17
```

#### （2）参数为对象

小规则：基本转为NaN，除非是包含单个数值的数组。

```JavaScript
Number({ a : 1 }) // NaN
Number([1, 2, 3 ]) // NaN
Number([5 ]) // 5
```

为什么会这样？ 到底发生了什么？

先来看这样一段代码。

代码1

```JavaScript
var obj = {
    
    toString: function(){
        return 1;
    },
    
    valueOf: function(){
        return 2;
    }
    
};
console.log(Number(obj))  // 2
```

结果为2，说明Number(obj)调用valueOf方法。

代码2

```JavaScript
var obj = {
    
    toString: function(){
        return 1;
    },
    
    valueOf: function(){
        return {};
    }
    
};
console.log(Number(obj)) // 1
```

结果为1，说明Number(obj)调用toString方法。

代码3

```JavaScript
var obj = {
    
    toString: function(){
        return {};
    },
    
    valueOf: function(){
        return {};
    }
    
};
console.log(Number(obj)) // 报错
```

如果toString方法返回的是对象，就报错。

代码4

```JavaScript
var obj = {
    
//     toString: function(){
//         return {};
//     },
    
    valueOf: function(){
        return {};
    }
    
};
console.log(Number(obj)) // NaN
```

相当于console.log(Number({})) // NaN
当我们注释掉obj内的toString方法，实际上Number(obj)也调用了 toString方法，调用的是Object.prototype上的toString方法。

```JavaScript
Number({}) 
// 可以拆解为
Object.prototype.toString.call({}) //"[object Object]"
Number("[object Object]") // NaN
```

代码5

```JavaScript
var obj = {
    
//     toString: function(){
//         return {};
//     },
    
    valueOf: function(){
        return [];
    }
    
};
console.log(Number(obj)) // 0
```

相当于console.log(Number([])) // 0

当我们注释掉obj内的toString方法，实际上Number(obj)也调用了 toString方法，首先Number的参数为[],首先看数组原型上有无toString方法，若有，直接调用，若没有，调用的是Object.prototype上的toString方法。

调用谁的toString方法，还是要看参数原型上有无toString方法，如果有，直接调用，如果没有，找参数原型的原型。

```JavaScript
Number([])
// 可以拆解为
Array.prototype.toString.call([]) // ""
Number("") // 0
```

注意一定分清楚具体调用的是谁的原型上toSting方法

Number([]) 若分析调用的是Object原型上toSting方法

```JavaScript
Object.prototype.toString.call([]) //"[object Array]"
Number("[object Array]") // NaN 
```

这样显然是不对的。

结论：

1. 调用对象自身的valueOf方法。如果返回原始类型的值，则直接对该值使用Number函数，不再进行后续步骤。

2. 如果valueOf方法返回的还是对象，则改为调用对象自身的toString方法。如果toString方法返回原始类型的值，则对该值使用Number函数，不再进行后续步骤。

3. 如果toString方法返回的是对象，就报错。

```JavaScript
  // 附：Object.prototype.toString 返回值 
  // 原始类型
  console.log(Object.prototype.toString.call('123')); // '[object String]'
  console.log(Object.prototype.toString.call(123)); // '[object Number]'
  console.log(Object.prototype.toString.call(NaN)); // '[object Number]'
  console.log(Object.prototype.toString.call(Infinity)); // '[object Number]'
  console.log(Object.prototype.toString.call(null)); // '[object Null]'
  console.log(Object.prototype.toString.call(undefined)); // '[object Undefined]'
  
  // 引用类型
  console.log(Object.prototype.toString.call(function() {})); // '[object Function]'
  console.log(Object.prototype.toString.call([1,2,3])); // '[object Array]'
  console.log(Object.prototype.toString.call([1])); // '[object Array]'
  console.log(Object.prototype.toString.call({})); // '[object Object]'

  console.log(Object.prototype.toString.call(new Error())); // '[object Error]'
  console.log(Object.prototype.toString.call(/\d/)); // '[object RegExp]'
  console.log(Object.prototype.toString.call(Date())); // '[object String]'
  console.log(Object.prototype.toString.call(new Date())); // '[object Date]'
  console.log(Object.prototype.toString.call(Symbol())); // '[object Symbol]'
  
  // 浏览器提供
  (function (){
    console.log(Object.prototype.toString.call(arguments)); // '[object Symbol]'
  })()
  console.log(Object.prototype.toString.call(document)); // '[object HTMLDocument]'
  console.log(Object.prototype.toString.call(window)); // '[object Window]'
```

### 1.2 String()

#### (1) 参数为原始类型

数值：转为相应的字符串

字符串：转换后还是原来的值

布尔值：true转为字符串"true"，false转为字符串"false"

undefined：转为字符串"undefined"

null：转为字符串"null"

```JavaScript
String(123) // "123"
String('abc') // "abc"
String(true) // "true"
String(undefined) // "undefined"
String(null) // "null"
```

#### (2) 参数为对象

```JavaScript
String({ a: 1 }) // "[object Object]"
String([1, 2, 3]) // "1,2,3"
```

```JavaScript
var obj = {
    toString: function(){
        return 1;
    },
    valueOf: function(){
        return 2;
    }
};
console.log(String(obj)
```

**String方法背后的转换规则，与Number方法基本相同，只是互换了valueOf方法和toString方法的执行顺序。**

1. 先调用对象自身的toString方法。如果返回原始类型的值，则对该值使用String函数，不再进行以下步骤。

2. 如果toString方法返回的是对象，再调用原对象的valueOf方法。如果valueOf方法返回原始类型的值，则对该值使用String函数，不再进行以下步骤。

3. 如果valueOf方法返回的是对象，就报错。

```JavaScript
[1, 2, 3].toString() === Array.prototype.toString.call([1, 2, 3]) // true

toString() // 实际上会根据类型去调用相应的toString方法。

Array.prototype.toString.call([1, 2]) == Object.prototype.toString.call([1, 2]) // false
// 不同类型的toString方法结果不同，要选择和其对应的toString方法。
```

### 1. 3 Boolean()

#### falsey(虚值)

```JavaScript
undefined null
0 NaN // +0  -0
false 
'' 
```

**除了虚值,其他都为true**

```JavaScript
Boolean([])
Boolean({}) 
Boolean(new Error()) 
Boolean(Symbol())
```

以上这些用排除法，全部为true

## 2.隐式类型转换/自动转换

### 2.1自动转换为布尔值

这些符号会自动转换为布尔值

```JavaScript
if()
for while switch
! !! 
===
? :
&& ||
```

隐式类型转换: **除了虚值,其他都为true**

### 2.2 自动转换为字符串

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
null+ [] ?
null + {} ?
null + funciton() {}
```

### 2.3自动转换为数值

**除了加法运算符（+）有可能把运算子转为字符串，其他运算符都会把运算子自动转成数值。**

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

null转为数值时为0，而undefined转为数值时为NaN。（特例）

#### 涉及数字转换

```JavaScript
+'abc' // NaN
-'abc' // NaN
+true // 1
-false // 0
```

```JavaScript
 + - * / %
++ --

涉及相等 都是number
> < == !== > < >= <+

[] == ![]
{} == !{}

// 特例
undefined == null // true
NaN == NaN // false

undefined === null // false
null === null // true
NaN === NaN // false
NaN == NaN // false

Number()
1 == true // true
'1'== true// true
'1' == 1 // true
[1] == 1 // true
[1] == true // true
[] == false // true
[] == null // false
[1, 2] == NaN // false

```

## 3.特殊值

### 特殊值:Infinity

```JavaScript
Number(Infinity) //Infinity
1/0 = Infinity 
1/Infinity = 0 
Infinity === Infinity // true
Infinity === -Infinity // false
0 === -0 // true
0/0 // NaN
```

```JavaScript
undefined == null // true
NaN == NaN // false
```

## 解题

```JavaScript
[] == ![]
// 可以拆解为
[].toSring  = ''
Number('') = 0

![] = false
Number(false) = 0

0 == 0  --->  [] == ![] 等于true

{} == !{}
// 可以拆解为
({}).toString() = "[object Object]";
Number("[object Object]") = NaN

!{} = false
Number(false) = 0

NaN != 0  --->  {} == !{} 等于false

```

## 拓展

```JavaScript
{} + {}
[] + {} 
{} + []

```
