---
title: JavaScript 代码整洁之道-函数篇
date: 2021-04-26 12:16:11
tags: JavaScript
---

> 本文源于翻译 [Clean Code Applied to JavaScript — Part III. Functions](https://dev.to/carlillo/clean-code-applied-to-javascript-part-iii-functions-4235)

# JavaScript 代码整洁之道-函数篇

在这篇文章中，我们将介绍书写整洁代码的基本技巧和建议，并重点关注于可复用的代码单元 -- **函数**。

本文中所有示例都是用 JavaScript 来实现，但是这些良好的实践适用于任何编程语言，包括 “最接近硬件”（[CTM](https://en.wikipedia.org/wiki/Close_to_Metal)） 的编程语言。为什么要特别强调这一点呢？曾经和同事讨论过，他们在工作中使用 C 或 Go 语言做开发，但是并不喜欢将这些实践应用于开发中，他们甚至认为在他们使用的编程语言中 “没有人” 会这样做。然后我的回答是，只有行动起来才能有所改变。尽管如此，我们还是对这些实践的优缺点做了较长时间的讨论。

接下来，我们开始介绍如何将这些技巧应用在具体的开发实践中。
## 使用默认参数去代替短路操作或条件赋值

在大多数编程语言中，函数的参数支持设置默认值。这就意味着我们可以在代码中避免使用短路操作和条件赋值。

下面的代码说明了这个例子。

```javascript
function setName(name) {
    const newName = name || 'Juan Palomo';
}
```

```javascript
function setName(name  = 'Juan Palomo') {
    // ...
}
```

## 函数参数(理想情况下不多于 2 个)

这条实践对于提高代码的质量至关重要，我们应该减少函数中入参的个数。理想情况下入参个数应该不多于 2 个，但也不要被这个数量所困扰，因为入参的数量还取决于我们使用的编程语言。

虽然这条建议非常重要，但是我们常常意识不到它的重要性。当一个函数有很多参数时，可以把这些参数组合在一起构成一个对象。我们需要避免使用多个基础类型 (如字符串、数字、布尔值等) 作为函数的入参，而是要使用抽象级别更高的对象作为入参。这样我们会更接近业务逻辑，并且更加远离底层实现。

第一个例子中，我们实现了一个生产汉堡的函数，它有 4 个参数。这些参数是固定的，并且必须按照这个顺序传参，这会给我们带来很多的限制。这样的函数在使用的时候不是很灵活。

第二个例子中，最大的改进就是使用一个对象来作为参数，只需要传入一个 `burger` 对象就可以生产出一个 “新汉堡”。通过这种方式，我们将汉堡的基本属性整合到 1 个对象里面。

在第三个例子中，我们对传入的对象进行解构赋值，让对象的属性在函数体中可访问到，但是实际上我们传入的仅仅是一个参数，这使得这个函数有了更大的灵活性。

```javascript
function newBurger(name, price, ingredients, vegan) {
    // ...
}

function newBurger(burger) {
    // ...
} 

function newBurger({ name, price, ingredients, vegan }) {
    // ...
} 
```

```javascript
const burger = {
    name: 'Chicken',
    price: 1.25,
    ingredients: ['chicken'],
    vegan: false,
};
newBurger(burger);

```

## 避免副作用 - 全局变量

副作用是未来麻烦的根源。虽然从定义上来说副作用不一定是有害的，但是如果在项目中没有节制的引起副作用，代码出错的可能性就会大大提高。

因此，本小节的建议是，不惜一切代价避免副作用，并且确保函数是可以被测试到的。下面的示例是一个经典的副作用场景，该函数修改了其作用域之外的变量。导致这个函数根本不能被测试，因为它没有要被测试的参数，造成这种情况的本质原因就是这个变量根本不受这个函数管控。

避免这种副作用的最简单方法是将此函数作用范围内的变量都作为参数进行传递。

```javascript
let fruits = 'Banana Apple';

function splitFruits() {
    fruits = fruits.split(' ');
}

splitFruits();

console.log(fruits); // ['Banana', 'Apple'];
```

```javascript
function splitFruits(fruits) {
    return fruits.split(' ');
}

const fruits = 'Banana Apple';
const newFruits = splitFruits(fruits);

console.log(fruits); // 'Banana Apple';
console.log(newFruits); // ['Banana', 'Apple'];
```

## 避免副作用 - 可变对象

另一个非常重要的副作用就是直接修改对象本身，如果你一直从事计算机相关的工作，你会知道 JavaScript 自诞生以来就是支持对象可变的，目前许多库都在尽量避免使用可变对象。

在 JavaScript 中，数组的方法一般被分为两部分: 一部分是会对数组本身进行修改的方法，例如: push、pop、sort，另一部分是不会对数组本身产生修改的方法例如：filter、reduce 、map 等。

如果您希望开发团队能够写出整洁、可维护的代码，您就必须制定特定代码规范来提高团队所有成员对代码和业务逻辑的理解。即使做这件事情可能会使我们的开发效率降低，但是这件事绝对是有意义的。

在这里给出两个例子，第一个是改变数组本身，第二个是返回一个新的数组。

```javascript
const addItemToCart = (cart, item) => {
    cart.push({ item, date: Date.now() });
}; 
```

```javascript
const addItemToCart = (cart, item) => {
    return [...cart, {
                item, 
                date: Date.now(),
            }];
};
```

## 函数应该只做一件事

这是所有计算机专业都会接触的编程原则之一，但在实践中，由于缺乏将理论付诸实践的能力，导致这些原则没有得到落实。每个函数只能专注于执行一个任务。当然在开发过程中肯定会有一些比较庞大的功能，它往往由很多的小任务构成，但是我们不应该把这些小的功能放在一个函数中编写，那样会造成代码的耦合。

因此，函数应该只做一件事。看下面这个例子，这个函数做的事情就是从客户列表中筛选出活跃的用户并给他们发送电子邮件。从概念上讲，这是一个简单的业务规则，但在实现它时，它们是两个明显独立的任务。

```javascript
function emailCustomers(customers) {
    customers.forEach((customer) => {
        const customerRecord = database.find(customer);
        if (customerRecord.isActive()) {
            email(client);
        }
    });
}
```

首先，我们必须过滤出活跃用户，这步操作就应该被封装成一个独立的函数。这里有一个地方要注意，当您在代码中准备写入 “if” 时，您应该考虑是否使用的恰当，虽然这并不意味着是错误的写法，但是在项目中滥用 “if” 肯定是不妥的行为。

当我们过滤完成得到了我们需要的用户之后，我们就需要另一个函数来向每个客户发送电子邮件。

```javascript
function emailActiveCustomers(customers) {
    customers
        .filter(isActiveCustomer)
        .forEach(email);
    }

function isActiveCustomer(customer) {
    const customerRecord = database.find(customer);
    return customerRecord.isActive();
}
```

请记住，您应该专注于每个函数只做一件事。

## 函数应该只是有一个抽象级别

设计函数时必须满足的另一个要求是，每个函数应仅具有单个抽象级别。

下面的示例显示的是一个工具函数。在这个函数中，包含了不同层次的抽象。

```javascript
function parseBetterJSAlternative(code) {
    const REGEXES = [
        // ...
    ];

    const statements = code.split(' ');
    const tokens = [];
    REGEXES.forEach((REGEX) => {
        statements.forEach((statement) => {
        // ...
        });
    });

    const ast = [];
    tokens.forEach((token) => {
        // lex...
    });

    ast.forEach((node) => {
        // parse...
    });
}                  

```

解决这个问题的方法非常简单，我们只需确定不同的抽象级别，并创建满足本文所述需求的函数。重构之后的函数如下：

```javascript
const REGEXES = [ // ...];
function tokenize(code) {    
    const statements = code.split(' ');
    const tokens = [];
    REGEXES.forEach((REGEX) => {
        statements.forEach((statement) => {
            tokens.push( /* ... */ );
        });
    });
    return tokens;
}
function lexer(tokens) {
    const ast = [];
    tokens.forEach((token) => ast.push( /* */ ));
    return ast;
}
function parseBetterJSAlternative(code) {
    const tokens = tokenize(code);
    const ast = lexer(tokens);
    ast.forEach((node) => // parse...);
}

```

## 优先考虑函数式编程而不是命令式编程

本文并不想引起编程范式间的争论，因为这不是写这篇文章的初衷。但是，还是还是希望大家学习函数式编程，并在命令式编程的中使用它。

我推荐阅读 [Alvin Alexander 的博客](https://alvinalexander.com/scala/fp-book/benefits-of-functional-programming)，在文章里介绍了函数式编程的一些好处。

下面，我总结了在命令中使用函数式编程的主要优点。

1. 纯函数更容易推理
2. 测试更容易，而且纯函数非常适合基于属性的测试
3. 调试更容易
4. 程序更加健壮
5. 程序是在更高的层次上编写的，因此更容易理解
6. 函数签名更有意义
7. 并行/并发编程更容易

函数式编程相对于命令式编程相比的另一个特点是代码更具可读性。如果你阅读了这一系列文章的第一篇，你会发现所有高质量代码都会有一个统一特征，那就是可读性强

函数式编程在这方面就具有很大的优势;但对于那些使用命令式编程学习并开始解决问题的初级程序员来说，他们很难使用这种编程范式，因为它改变了他们的工作习惯。但是在这个行业中，我们必须适应变化，况且目前使用函数式编程的场景越来越多。

阅读这段统计商品总价格的代码，您肯定需要记住这几个变量: total，i，items，items.length，price; 而在下面的函数式编程实现中，我们只需要三个变量 total，price 和 items。如果您习惯于使用功能性操作符，那么它的读取速度非常快，而且非常友好。

```javascript
const items = [{
    name: 'Coffe',
    price: 500
  }, {
    name: 'Ham',
    price: 1500
  }, {
    name: 'Bread',
    price: 150
  }, {
    name: 'Donuts',
    price: 1000
  }
];
```

```javascript
let total = 0;
for (let i = 0; i < items.length; i++) {
  total += items[i].price;
}
```

```javascript
const total = items
  .map(({ price }) => price)
  .reduce((total, price) => total + price);
```

## 函数链式调用

我们设计用于操作对象或数据流（在本例中是对象）的函数，通常是执行单个任务的函数。在复杂的场景下，会把这些方法组合起来使用，并且不能引起副作用。这时使用函数的链式调用方式会使代码的可读性更好。

如果你了解 Linux，你会发现它的每一个命令都旨在做一件事情并且都很好地完成工作，但是操作系统往往都很复杂，这是通过使用管道来组合不同的简单命令来实现的。

在具体案例中，我们也必须使用类似的做法，无论是使用对象还是函数中。下面将分别展示使用传统方法和链式方法的 Car 。

```javascript
class Car {
    constructor({ make, model, color } = car) {
        /* */
    }
    setMake(make) {
        this.make = make;
    }
    setModel(model) {
        this.model = model;
    }
    setColor(color) {
        this.color = color;
    }
    save() {
        console.log(this.make, this.model, this.color);
    }
}    
const car = new Car('WV','Jetta','gray');
car.setColor('red');
car.save();
```

```javascript
class Car {
    constructor({ make, model, color } = car){}
    setMake(make) {
        this.make = make;
        return this;
    }
    setModel(model) {
        this.model = model;
        return this;
    }
    setColor(color) {
        this.color = color;
        return this;
    }
    save() {
        console.log(this.make, this.model, this.color);
        return this;
    }
}
const car = new Car('WV','Jetta','gray')
.setColor('red')
.save();
```

## 结论

在这篇文章中，我们讨论了开发人员如何将整洁的代码应用于各种编程语言中的基础单元：函数。

实际上，使用整洁的代码去编写函数是十分重要的，因为函数本身就是为了解耦代码而存在的。但是可能因为我们开发过程中的一些不良习惯导致函数代码中仍然有很多耦合存在，此外，糟糕的函数设计会导致一些难以发现的严重错误。随着我们项目的抽象层次的提高，定位错误就会变得更加困难。

因此，这篇文章中的建议会让你的代码质量提升一个级别，但是在没有充分思考的情况下不要应用它们。请记住，这里没有魔法提示或银弹，但有一套技术可以让你解决更广泛的问题。



最后，我们讨论的要点如下:

+ 使用默认参数而不是短路操作或条件赋值
+ 函数参数(最好不多于 2 个)
+ 避免副作用 - 全局变量
+ 避免副作用 - 可变对象
+ 函数应该做一件事
+ 函数应该只有一个抽象级别
+ 优先考虑函数式编程而不是命令式编程