---
title: JavaScript 代码整洁之道-复杂判断
date: 2021-05-10 18:37:55
tags: JavaScript
---

> 本文源于翻译[Clean Code Applied to JavaScript — Part VI. Avoid Conditional Complexity](https://dev.to/carlillo/clean-code-applied-to-javascript-part-vi-avoid-conditional-complexity-2j8i)

# JavaScript 代码整洁之道-复杂判断

复杂的条件判断会使代码难于理解，因而增加了维护成本。此外，条件的复杂程度通常也是衡量代码耦合程度的指标之一。如果想要提高代码质量，最好避免编写复杂条件判断的代码。

在这篇文章中，将介绍一些在代码中避免复杂条件的方法和建议。在下面具体的例子中，我们将使以 `JavaScript/TypeScript` 语言为例，但在这篇文章中讨论的概念可以应用于任何编程语言，因为我们提供的建议和方法技巧不局限于某一个特定的编程语言。

## 不要使用标记 (flag) 作为函数参数

避免复杂条件的第一个建议是不要将标记作为函数参数，否则会破坏函数功能的单一性。我们必须创建两个函数来实现各自对应的逻辑功能，而不是使用一个函数来实现两个逻辑功能，因为它们是不同的功能。

在第一个示例中通过 `isPremium` 参数来决定使用哪个函数。然而，正确的方法应该是声明两个不同的函数来实现这两个功能。

```js
// Dirty
function book(customer, isPremium) {
  // ...
  if (isPremium) {
    premiumLogic();
  } else {
    regularLogic();
  }
}
```

```js
// Clean (Declarative way)
function bookPremium (customer) {
  premiumLogic();
}

function bookRegular (customer) {
  retularLogic();
}
```

## 封装判断条件

请将条件封装在具有明确语义的函数中，这样可以更直观的理解代码逻辑。在第一个例子中，需要大家对该条件语句做一定的思考才能理解，而第二个例子通过函数名就可以直接理解。

```js
if (platform.state === 'fetching' && isEmpty(cart)) {
    // ...
}  
```

```js
function showLoading(platform, cart) {
    return platform.state === 'fetching' && isEmpty(cart);
}

if (showLoading(platform, cart)) {
    // ...
}
```

## 用卫语句 (Guard Clauses) 替换嵌套的条件语句

这个建议在程序员的开发中是至关重要的，在开发中不应该有嵌套的条件语句存在。卫语句是我们避免嵌套条件的主要技术之一，不需要 `else` 关键字就可以完美地实现。

下面的示例演示了代码中的卫语句，通过编辑器的自动格式化，代码的可读性已经得到了很大程度的提升。

如果你想深入了解卫语句，我建议你阅读我具体的文章：[Guard Clauses](https://medium.com/better-programming/refactoring-guard-clauses-2ceeaa1a9da).


```js
function getPayAmount() {
    let result;
    if (isDead){
        result = deadAmount();
    } else {
        if (isSeparated){
            result = separatedAmount();
        } else {
            if (isRetired){
                result = retiredAmount();
            }else{
                result = normalPayAmount();
            }
        }
    }
    return result;
}
```

```js
function getPayAmount() {
    if (isDead) return deadAmount();
    if (isSeparated) return separatedAmount();
    if (isRetired) return retiredAmount();
    return normalPayAmount();
}
```

## 空对象 (Null Object) 模式

在初级程序员的代码中可以看到的另一个常见错误，不断检查对象是否为 `null`，并根据该检查判断是否显示默认操作。这种模式称为空对象模式。

在下面的示例中，检查数组中的每个对象，如果 `animal` 是 `null` 则不执行 `sound` 方法。

另一方面，如果我们创建一个封装了空对象行为的对象，我们将不再需要执行第一段代码中所展示的那样的验证。

```js
class Dog {
  sound() {
    return 'bark';
  }
}

['dog', null].map((animal) => {
  if(animal !== null) { 
    sound(); 
  }
 });

```

```js
class Dog {
  sound() {
    return 'bark';
  }
}

class NullAnimal {
  sound() {
    return null;
  }
}

function getAnimal(type) {
  return type === 'dog' ? new Dog() : new NullAnimal();
}

['dog', null].map((animal) => getAnimal(animal).sound());
// Returns ["bark", null]
```

如果您想深入了解这种模式，我建议您阅读我的具体文章：[Null-Object Pattern](https://medium.com/better-programming/design-patterns-null-object-5ee839e37892)

## 使用多态删除条件

绝大多数程序员会认为 `switch` 控制语句要比 `if` 语句更简洁，虽然两者的用法有一定的不同。如果我们在代码中使用了 `switch`，其实也会提高代码的复杂性，最终会让我们思考得太多。

下面的示例中，通过判断对象的类型去定义对象的方法，但是这里的条件语句被滥用了。在这种场景下，我们可以通过类的继承，为每个特定类型创建一个类，利用多态来避免使用条件判断。

```js
function Auto() {
}
Auto.prototype.getProperty = function () {
    switch (type) {
        case BIKE:
            return getBaseProperty();
        case CAR:
            return getBaseProperty() - getLoadFactor();
        case BUS:
            return (isNailed) ? 
            0 : 
            getBaseProperty(voltage);
    }
    throw new Exception("Should be unreachable");
};
```

```js
abstract class Auto { 
    abstract getProperty();
}

class Bike extends Auto {
    getProperty() {
        return getBaseProperty();
    }
}
class Car extends Auto {
    getProperty() {
        return getBaseProperty() - getLoadFactor();
    }
}
class Bus extends Auto {
    getProperty() {
        return (isNailed) ? 
                0 : 
                getBaseProperty(voltage);
    }
}
// Somewhere in client code
speed = auto.getProperty();
```

## 使用策略模式/命令模式移除条件

我们在代码中也可以使用策略模式或命令模式来避免复杂条件。

如果您想深入了解这两种模式，我建议您阅读我在这些模式中深入研究的具体文章：[Strategy Pattern](https://medium.com/better-programming/design-patterns-using-the-strategy-pattern-in-javascript-3c12af58fd8a) 和 [Command Pattern](https://medium.com/better-programming/the-command-design-pattern-2313909122b5)。

在本节的具体示例中，你可以了解动态选择策略的策略模式。注意这里是如何使用不同的策略来解决 `switch` 结构的复杂性这个问题的。

```js
function logMessage(message = "CRITICAL::The system ..."){
    const parts = message.split("::"); 
    const level = parts[0];

    switch (level) {
        case 'NOTICE':
            console.log("Notice")
            break;
        case 'CRITICAL':
            console.log("Critical");
            break;
        case 'CATASTROPHE':
           console.log("Castastrophe");
            break;
    }
}
```

```js
const strategies = {
    criticalStrategy,
    noticeStrategy,
    catastropheStrategy,
}
function logMessage(message = "CRITICAL::The system ...") {
    const [level, messageLog] = message.split("::");
    const strategy = `${level.toLowerCase()}Strategy`;
    const output = strategies[strategy](messageLog);
}
function criticalStrategy(param) {
    console.log("Critical: " + param);
}
function noticeStrategy(param) {
    console.log("Notice: " + param);
}
function catastropheStrategy(param) {
    console.log("Catastrophe: " + param);
}
logMessage();
logMessage("CATASTROPHE:: A big Catastrophe");
```

## 总结

在本文中，我们提出了一些避免复杂条件判断的建议。复杂的条件判断会使代码可读性变差，也会使代码高度耦合，导致灵活度降低。这些方法和建议让我们在代码中避免使用复杂判断，使我们的代码质量再上一个台阶。

最后，我们总结了以下几点：

- 不要将标志 (flag) 用作函数参数
- 封装判断条件
- 用卫语句 (Guard Clauses) 替换嵌套的条件语句
- 空对象 (Null Object) 模式
- 使用多态避免条件判断
- 使用策略模式避免条件判断
- 使用命令模式避免条件判断
