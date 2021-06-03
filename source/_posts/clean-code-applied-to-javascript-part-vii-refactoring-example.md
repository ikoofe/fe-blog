---
title: JavaScript 代码整洁之道-重构篇
date: 2021-06-03 15:18:11
tags: JavaScript
---

> 本文源于翻译 [Clean Code Applied to JavaScript - Part VII: Practical Refactoring Example: Ceaser Cipher ](https://dev.to/carlillo/clean-code-applied-to-javascript-part-vii-practical-refactoring-example-ceaser-cipher-2397)

# JavaScript 代码整洁之道 - 重构篇

## 引言

在本系列文章中，我们介绍了一些编写高可维护性代码的技巧。这些编程技巧和建议来自于《[代码整洁之道](https://book.douban.com/subject/4199741/)》一书，以及多年来我对这些技巧的应用经验。

在本文中，将介绍如何对代码一步一步地实现重构，并且以我在编程基础课程中已经实现的代码为例，来展示如何在实践中应用这些技巧。如果你刚刚接触或学习软件开发，那么建议你使用熟悉的技术和工具来练习和理解本文中的代码示例（本文使用 JavaScript 作为编程语言）。如果你已经具备一定的编程基础，相对来说会比较容易理解接下来要介绍的重构案例。在这个案例中已经存在了一种最初的代码实现方案，它的挑战在于应用不同的重构技术来深入理解代码，并使代码更易于维护。

对于这个挑战，我准备了一个 GIT 仓库，你可以找到我们在本文中每一步重构的算法的所有版本。通过一系列 npm 脚本，
可以执行每一个重构步骤中的代码。具体的脚本命令如下所示：

```bash
npm run stepX # Where X is the step
```

相关代码的 GIT 仓库：[代码仓库链接](https://github.com/Caballerog/blog/tree/master/refactoring-caesar)。

## 问题：恺撒密码

恺撒密码 (Ceaser Cipher) 是一种最简单且最广为人知的加密技术。它是一种替换加密的技术，明文中的所有字母都在字母表上向左（或向右）按照一个固定数目进行偏移后被替换成密文。例如，当右的偏移量为 3 的时候，所有的字母 E 将被替换成 H，F 变成了 I，以此类推。可以进入[维基百科](https://en.wikipedia.org/wiki/Caesar_cipher)查看更多关于恺撒密码的介绍。

恺撒密码是由明文和密文两个字母表构成的，密文字母表是将明文字母表向左或向右移动一个固定位置得到的。例如，这个恺撒密码，使用的偏移量为 6，相当于右移 6 位：

```
Plain:    ABCDEFGHIJKLMNOPQRSTUVWXYZ
Cipher:   GHIJKLMNOPQRSTUVWXYZABCDEF 
```

加密时，找到明文中的每一个字母在明文字母表中的位置，并且记录下密文字母表中该位置所对应的字母。

Plaintext: THE QUICK BROWN FOX JUMPS OVER THE LAZY DOG

Ciphertext: QEB NRFZH YOLTK CLU GRJMP LSBO QEB IXWV ALD

解密则是反向进行的，左移 6 个位置。

## 什么是重构？为什么重构？

重构在软件开发行业中是一个众所周知的主题。对此，我们对该主题进行了介绍，但我建议你阅读下面的文章：[https://www.cuelogic.com/blog/what-is-refactoring-and-why-is-it-important](https://www.cuelogic.com/blog/what-is-refactoring-and-why-is-it-important)。从这篇文章中，我们提取了我们将要在这里分享的主要观点。

> 重构不是银弹，但它是一种有价值的武器，可以帮助你控制好代码和项目 (软件/应用)。

> 它是一个科学的过程，对现有的代码进行改造，使代码可读性更高、更好理解和更整洁。而且，使添加新功能、构建大型应用程序以及发现和修复 bug 变得非常便捷。

重构很重要的原因：

- 改进软件/应用程序的设计
- 使软件更容易理解
- 发现 bug
- 修复现有的旧数据库
- 为用户提供更好的一致性

## 初始代码

一旦我们知道了想要解决的问题，我们就会实现一个任何人都可以在很少的时间内完成的方案。

```JavaScript
function cipher(text, shift) {
  var cipher = '';
  shift = shift % 26;
  for (var i = 0; i < text.length; i++) {
    if (text.charCodeAt(i) >= 65 && text.charCodeAt(i) <= 90) {
      if (text.charCodeAt(i) + shift > 90) {
        cipher = cipher.concat(
          String.fromCharCode(text.charCodeAt(i) + shift - 26),
        );
      } else {
        cipher = cipher.concat(String.fromCharCode(text.charCodeAt(i) + shift));
      }
    } else if (text.charCodeAt(i) >= 97 && text.charCodeAt(i) <= 122) {
      if (text.charCodeAt(i) + shift > 122) {
        cipher = cipher.concat(
          String.fromCharCode(text.charCodeAt(i) + shift - 26),
        );
      } else {
        cipher = cipher.concat(String.fromCharCode(text.charCodeAt(i) + shift));
      }
    } else {
      // blank space
      cipher = cipher.concat(String.fromCharCode(text.charCodeAt(i) + shift));
    }
  }
  return cipher.toString();
}

function decipher(text, shift) {
  var decipher = '';
  shift = shift % 26;
  for (var i = 0; i < text.length; i++) {
    if (text.charCodeAt(i) >= 65 && text.charCodeAt(i) <= 90) {
      if (text.charCodeAt(i) - shift < 65) {
        decipher = decipher.concat(
          String.fromCharCode(text.charCodeAt(i) - shift + 26),
        );
      } else {
        decipher = decipher.concat(
          String.fromCharCode(text.charCodeAt(i) - shift),
        );
      }
    } else if (text.charCodeAt(i) >= 97 && text.charCodeAt(i) <= 122) {
      if (text.charCodeAt(i) - shift < 97) {
        decipher = decipher.concat(
          String.fromCharCode(text.charCodeAt(i) - shift + 26),
        );
      } else {
        decipher = decipher.concat(
          String.fromCharCode(text.charCodeAt(i) - shift),
        );
      }
    } else {
      // blank space
      decipher = decipher.concat(
        String.fromCharCode(text.charCodeAt(i) - shift),
      );
    }
  }
  return decipher.toString();
}

```

我们要开发的代码有两个方法:
- `cipher` - 它将文本按一个方向移动。 
- `decipher` - 执行与 `cipher` 相反的操作，也就是破译文本。

我建议，无论我们如何重构代码，都要使用自动化测试来帮助我们验证有没有 “破坏” 代码。在这个例子中，我没有实现一整套测试方案，而是使用标准的 `console.assert` 创建了两个检查。

因此，我们将通过以下断言来检查算法是否稳定。

```JavaScript
console.assert(
  cipher('Hello World', 1) === 'Ifmmp!Xpsme',
  `${cipher('Hello World', 1)} === 'Ifmmp!Xpsme'`,
);
console.assert(
  decipher(cipher('Hello World', 3), 3) === 'Hello World',
  `${decipher(cipher('Hello World', 3), 3)} === 'Hello World'`,
);

```

现在让我们迎接挑战吧，开始重构吧！

## Step 1. 魔法数字

第一步是通过定义语义化的变量来移除代码中出现的魔法数字。可以这样修改以下数字:

1. 字母表中的字母数 (26)
2. 算法循环中每一个字母的临界值，即：
- a: 65
- z: 90
- A: 97
- Z: 122

因此，我们定义了以下常量，这些常量使这些数字具有语义化。

```JavaScript
const NUMBER_LETTERS = 26;
const LETTER = {
  a: 65,
  z: 90,
  A: 97,
  Z: 122,
};

```

通过第一步更改之后，代码就变成了这样。

```JavaScript
const NUMBER_LETTERS = 26;
const LETTER = {
  a: 65,
  z: 90,
  A: 97,
  Z: 122,
};

function cipher(text, shift) {
  let cipher = '';
  shift = shift % NUMBER_LETTERS;
  for (let i = 0; i < text.length; i++) {
    if (text.charCodeAt(i) >= LETTER.a && text.charCodeAt(i) <= LETTER.z) {
      if (text.charCodeAt(i) + shift > LETTER.z) {
        cipher = cipher.concat(
          String.fromCharCode(text.charCodeAt(i) + shift - NUMBER_LETTERS),
        );
      } else {
        cipher = cipher.concat(String.fromCharCode(text.charCodeAt(i) + shift));
      }
    } else if (
      text.charCodeAt(i) >= LETTER.A &&
      text.charCodeAt(i) <= LETTER.Z
    ) {
      if (text.charCodeAt(i) + shift > LETTER.Z) {
        cipher = cipher.concat(
          String.fromCharCode(text.charCodeAt(i) + shift - NUMBER_LETTERS),
        );
      } else {
        cipher = cipher.concat(String.fromCharCode(text.charCodeAt(i) + shift));
      }
    } else {
      // blank space
      cipher = cipher.concat(String.fromCharCode(text.charCodeAt(i) + shift));
    }
  }
  return cipher.toString();
}

function decipher(text, shift) {
  let cipher = '';
  shift = shift % NUMBER_LETTERS;
  for (let i = 0; i < text.length; i++) {
    if (text.charCodeAt(i) >= LETTER.a && text.charCodeAt(i) <= LETTER.z) {
      if (text.charCodeAt(i) - shift < LETTER.a) {
        cipher = cipher.concat(
          String.fromCharCode(text.charCodeAt(i) - shift + NUMBER_LETTERS),
        );
      } else {
        cipher = cipher.concat(String.fromCharCode(text.charCodeAt(i) - shift));
      }
    } else if (
      text.charCodeAt(i) >= LETTER.A &&
      text.charCodeAt(i) <= LETTER.Z
    ) {
      if (text.charCodeAt(i) - shift < LETTER.A) {
        cipher = cipher.concat(
          String.fromCharCode(text.charCodeAt(i) - shift + NUMBER_LETTERS),
        );
      } else {
        cipher = cipher.concat(String.fromCharCode(text.charCodeAt(i) - shift));
      }
    } else {
      cipher = cipher.concat(String.fromCharCode(text.charCodeAt(i) - shift));
    }
  }
  return cipher.toString();
}

console.assert(
  cipher('Hello World', 1) === 'Ifmmp!Xpsme',
  `${cipher('Hello World', 1)} === 'Ifmmp!Xpsme'`,
);
console.assert(
  decipher(cipher('Hello World', 3), 3) === 'Hello World',
  `${decipher(cipher('Hello World', 3), 3)} === 'Hello World'`,
);

```

## Step 2. 从 if-else 中提取相似代码

下一步是将代码中重复的代码提取到函数中。具体来说，`if` 控制结构体中的赋值代码在整个代码中是重复的，我们可以提取这些赋值代码。

也就是说，这个代码片段 `cipher=cipher.concat（String.fromCharCode）` 可以从代码中不同 `if` 中提取出来。这一行是在 `if` 结构之后执行的，而 `if` 结构只包含每个情况下的不同逻辑。

当然，我们对 `cipher` 函数执行的操作也适用于 `decipher` 函数。

应用此重构后的代码如下:


```JavaScript
function cipher(text, shift) {
  let cipher = '';
  shift = shift % NUMBER_LETTERS;

  for (let i = 0; i < text.length; i++) {
    let character = '';
    if (text.charCodeAt(i) >= LETTER.a && text.charCodeAt(i) <= LETTER.z) {
      if (text.charCodeAt(i) + shift > LETTER.z) {
        character = text.charCodeAt(i) + shift - NUMBER_LETTERS;
      } else {
        character = text.charCodeAt(i) + shift;
      }
    } else if (
      text.charCodeAt(i) >= LETTER.A &&
      text.charCodeAt(i) <= LETTER.Z
    ) {
      if (text.charCodeAt(i) + shift > LETTER.Z) {
        character = text.charCodeAt(i) + shift - NUMBER_LETTERS;
      } else {
        character = text.charCodeAt(i) + shift;
      }
    } else {
      // blank space
      character = text.charCodeAt(i) + shift;
    }
    cipher = cipher.concat(String.fromCharCode(character));
  }
  return cipher.toString();
}
function decipher(text, shift) {
  let cipher = '';

  shift = shift % NUMBER_LETTERS;
  for (let i = 0; i < text.length; i++) {
    let character = '';
    if (text.charCodeAt(i) >= LETTER.a && text.charCodeAt(i) <= LETTER.z) {
      if (text.charCodeAt(i) - shift < LETTER.a) {
        character = text.charCodeAt(i) - shift + NUMBER_LETTERS;
      } else {
        character = text.charCodeAt(i) - shift;
      }
    } else if (
      text.charCodeAt(i) >= LETTER.A &&
      text.charCodeAt(i) <= LETTER.Z
    ) {
      if (text.charCodeAt(i) - shift < LETTER.A) {
        character = text.charCodeAt(i) - shift + NUMBER_LETTERS;
      } else {
        character = text.charCodeAt(i) - shift;
      }
    } else {
      character = text.charCodeAt(i) + shift;
    }
    cipher = cipher.concat(String.fromCharCode(character));
  }
  return cipher.toString();
}
```

## Step 3. 避免使用 else

下一步是避免使用 `else` 控制结构。避免使用它是很容易的，我们只需要在循环开始之前将代码从 `else` 移到变量 `character`，并作为它的默认值。

因此，重构后的代码如下：

```JavaScript
function cipher(text, shift) {
  let cipher = '';
  shift = shift % NUMBER_LETTERS;
  for (let i = 0; i < text.length; i++) {
    let character = text.charCodeAt(i) + shift;
    if (text.charCodeAt(i) >= LETTER.a && text.charCodeAt(i) <= LETTER.z) {
      if (text.charCodeAt(i) + shift > LETTER.z) {
        character = text.charCodeAt(i) + shift - NUMBER_LETTERS;
      } else {
        character = text.charCodeAt(i) + shift;
      }
    } else if (
      text.charCodeAt(i) >= LETTER.A &&
      text.charCodeAt(i) <= LETTER.Z
    ) {
      if (text.charCodeAt(i) + shift > LETTER.Z) {
        character = text.charCodeAt(i) + shift - NUMBER_LETTERS;
      } else {
        character = text.charCodeAt(i) + shift;
      }
    }
    cipher = cipher.concat(String.fromCharCode(character));
  }
  return cipher.toString();
}
function decipher(text, shift) {
  let cipher = '';

  shift = shift % NUMBER_LETTERS;
  for (let i = 0; i < text.length; i++) {
    let character = text.charCodeAt(i) + shift;
    if (text.charCodeAt(i) >= LETTER.a && text.charCodeAt(i) <= LETTER.z) {
      if (text.charCodeAt(i) - shift < LETTER.a) {
        character = text.charCodeAt(i) - shift + NUMBER_LETTERS;
      } else {
        character = text.charCodeAt(i) - shift;
      }
    } else if (
      text.charCodeAt(i) >= LETTER.A &&
      text.charCodeAt(i) <= LETTER.Z
    ) {
      if (text.charCodeAt(i) - shift < LETTER.A) {
        character = text.charCodeAt(i) - shift + NUMBER_LETTERS;
      } else {
        character = text.charCodeAt(i) - shift;
      }
    }
    cipher = cipher.concat(String.fromCharCode(character));
  }
  return cipher.toString();
}
```

## Step 4. 合并 IF 逻辑

下一步对我们来说是麻烦的，但我们必须合并与 `if-elseif` 对应的逻辑。所以，我们只需要有两个 `if` 结构。这个操作会使我们在之后的步骤中看到，我们确实有两条可选择的路径，而不是现在我们看到的这些。

合并后的 `if` 逻辑代码如下：

```JavaScript
function cipher(text, shift) {
  let cipher = '';
  shift = shift % NUMBER_LETTERS;
  for (let i = 0; i < text.length; i++) {
    let character = text.charCodeAt(i) + shift;
    if (
      (text.charCodeAt(i) >= LETTER.a &&
        text.charCodeAt(i) <= LETTER.z &&
        text.charCodeAt(i) + shift > LETTER.z) ||
      (text.charCodeAt(i) >= LETTER.A &&
        text.charCodeAt(i) <= LETTER.Z &&
        text.charCodeAt(i) + shift > LETTER.Z)
    ) {
      character = text.charCodeAt(i) + shift - NUMBER_LETTERS;
    }
    if (
      (text.charCodeAt(i) >= LETTER.a &&
        text.charCodeAt(i) <= LETTER.z &&
        text.charCodeAt(i) + shift > LETTER.z &&
        !(text.charCodeAt(i) >= LETTER.a && text.charCodeAt(i) <= LETTER.z)) ||
      (text.charCodeAt(i) >= LETTER.A &&
        text.charCodeAt(i) <= LETTER.Z &&
        !(text.charCodeAt(i) + shift > LETTER.Z))
    ) {
      character = text.charCodeAt(i) + shift;
    }
    
    cipher = cipher.concat(String.fromCharCode(character));
  }
  return cipher.toString();
}

function decipher(text, shift) {
  let cipher = '';

  shift = shift % NUMBER_LETTERS;
  for (let i = 0; i < text.length; i++) {
    let character = text.charCodeAt(i) - shift;
    if (
      (text.charCodeAt(i) >= LETTER.a &&
        text.charCodeAt(i) <= LETTER.z &&
        text.charCodeAt(i) - shift < LETTER.a) ||
      (text.charCodeAt(i) >= LETTER.A &&
        text.charCodeAt(i) <= LETTER.Z &&
        text.charCodeAt(i) - shift < LETTER.A)
    ) {
      character = text.charCodeAt(i) - shift + NUMBER_LETTERS;
    }
    if (
      (text.charCodeAt(i) >= LETTER.a &&
        text.charCodeAt(i) <= LETTER.z &&
        !(text.charCodeAt(i) - shift < LETTER.a)) ||
      (text.charCodeAt(i) >= LETTER.A &&
        text.charCodeAt(i) <= LETTER.Z &&
        !(text.charCodeAt(i) - shift < LETTER.A))
    ) {
      character = text.charCodeAt(i) - shift;
    }
    cipher = cipher.concat(String.fromCharCode(character));
  }
  return cipher.toString();
}
```

## Step 5. 简化算法逻辑

在这一步中，我们必须说明我们的算法不需要两个 `if` 结构。相反，`cipher` 和 `decipher` 函数都有 `if-else` 结构。对于函数`cipher`，我们看到有两种值可能会赋给变量 `character`，第一种可能的值是从相应的第一个 `if` 得到的。

```JavaScript
character = text.charCodeAt(i) + shift - NUMBER_LETTERS;
```

第二个可能的值是在默认情况下和从另一个 `if` 结构中获得的:

```JavaScript
character = text.charCodeAt(i) + shift;
```

因此，可以删除第二个 `if` 的逻辑，并将控制结构转换为与第一个 `if` 结构对应的 `else`，在这个 `if` 的条件不满足的情况下，第二个可能的值将会赋值给变量 `character`。不管第二个 `if` 是否满足，都会由默认值赋值。

重构后的代码如下：

```JavaScript
function cipher(text, shift) {
  let cipher = '';
  shift = shift % NUMBER_LETTERS;
  for (let i = 0; i < text.length; i++) {
    let character;
    if (
      (text.charCodeAt(i) >= LETTER.a &&
        text.charCodeAt(i) <= LETTER.z &&
        text.charCodeAt(i) + shift > LETTER.z) ||
      (text.charCodeAt(i) >= LETTER.A &&
        text.charCodeAt(i) <= LETTER.Z &&
        text.charCodeAt(i) + shift > LETTER.Z)
    ) {
      character = text.charCodeAt(i) + shift - NUMBER_LETTERS;
    } else {
      character = text.charCodeAt(i) + shift;
    }

    cipher = cipher.concat(String.fromCharCode(character));
  }
  return cipher.toString();
}

function decipher(text, shift) {
  let cipher = '';

  shift = shift % NUMBER_LETTERS;
  for (let i = 0; i < text.length; i++) {
    if (
      (text.charCodeAt(i) >= LETTER.a &&
        text.charCodeAt(i) <= LETTER.z &&
        text.charCodeAt(i) - shift < LETTER.a) ||
      (text.charCodeAt(i) >= LETTER.A &&
        text.charCodeAt(i) <= LETTER.Z &&
        text.charCodeAt(i) - shift < LETTER.A)
    ) {
      character = text.charCodeAt(i) - shift + NUMBER_LETTERS;
    } else {
      character = text.charCodeAt(i) - shift;
    }
    cipher = cipher.concat(String.fromCharCode(character));
  }
  return cipher.toString();
}
```


## Step 6. 封装条件

由于缺乏语义值，使得我们的算法条件相当复杂并且难以理解。所以，代码中的下一步是封装条件。

具体来说，我们专注于封装 `cipher` 和 `decipher` 条件：

cipher:

```JavaScript
(text.charCodeAt(i) >= LETTER.a && text.charCodeAt(i) <= LETTER.z && text.charCodeAt(i) + shift > LETTER.z) 
||
(text.charCodeAt(i) >= LETTER.A && text.charCodeAt(i) <= LETTER.Z && text.charCodeAt(i) + shift > LETTER.Z)
```

decipher:

```JavaScript
(text.charCodeAt(i) >= LETTER.a && text.charCodeAt(i) <= LETTER.z && text.charCodeAt(i) - shift < LETTER.a) 
||
(text.charCodeAt(i) >= LETTER.A && text.charCodeAt(i) <= LETTER.Z && text.charCodeAt(i) - shift < LETTER.A)
```

实际上，这个逻辑可以概括为以下四个函数：

```JavaScript
function isOutLowerCharacterCipher(text, position, shift) {
  return (
    text.charCodeAt(position) >= LETTER.a &&
    text.charCodeAt(position) <= LETTER.z &&
    text.charCodeAt(position) + shift > LETTER.z
  );
}

function isOutUpperCharacterCipher(text, position, shift) {
  return (
    text.charCodeAt(position) >= LETTER.A &&
    text.charCodeAt(position) <= LETTER.Z &&
    text.charCodeAt(position) + shift > LETTER.Z
  );
}

function isOutLowerCharacterDecipher(text, position, shift) {
  return (
    text.charCodeAt(position) >= LETTER.a &&
    text.charCodeAt(position) <= LETTER.z &&
    text.charCodeAt(position) - shift < LETTER.a
  );
}

function isOutUpperCharacterDecipher(text, position, shift) {
  return (
    text.charCodeAt(position) >= LETTER.A &&
    text.charCodeAt(position) <= LETTER.Z &&
    text.charCodeAt(position) - shift < LETTER.A
  );
}
```

执行此封装后的代码如下：

```JavaScript
function cipher(text, shift) {
  let cipher = '';
  shift = shift % NUMBER_LETTERS;
  for (let i = 0; i < text.length; i++) {
    let character;
    if (
      isOutLowerCharacterCipher(text, i, shift) ||
      isOutUpperCharacterCipher(text, i, shift)
    ) {
      character = text.charCodeAt(i) + shift - NUMBER_LETTERS;
    } else {
      character = text.charCodeAt(i) + shift;
    }

    cipher = cipher.concat(String.fromCharCode(character));
  }
  return cipher.toString();
}

function decipher(text, shift) {
  let cipher = '';

  shift = shift % NUMBER_LETTERS;
  for (let i = 0; i < text.length; i++) {
    if (
      isOutLowerCharacterDecipher(text, i, shift) ||
      isOutUpperCharacterDecipher(text, i, shift)
    ) {
      character = text.charCodeAt(i) - shift + NUMBER_LETTERS;
    } else {
      character = text.charCodeAt(i) - shift;
    }
    cipher = cipher.concat(String.fromCharCode(character));
  }
  return cipher.toString();
}
```

## Step 7.  移除 if-else 控制结构

控制结构`if else` 都对同一变量（`character`）进行赋值。因此，你可以从 `if` 中提取条件的逻辑并存储在变量中，如下所示：

```JavaScript
const isOutAlphabet =
      isOutLowerCharacterCipher(text, i, shift) ||
      isOutUpperCharacterCipher(text, i, shift);
```

变量`character`是由具有两个可能的值交替修改的：
1. `NUMBER_LETTERS`
2. 0 (`NO_ROTATION`);

因此，我们可以定义变量`rotation`，这样就可以提高代码的粒度，如下所示：

```JavaScript
const rotation = isOutAlphabet ? NUMBER_LETTERS : NO_ROTATION;

```

结果代码如下：

```JavaScript
const isOutAlphabet =
isOutLowerCharacterCipher(text, i, shift) ||
isOutUpperCharacterCipher(text, i, shift);
const rotation = isOutAlphabet ? NUMBER_LETTERS : NO_ROTATION;
const character = text.charCodeAt(i) + shift - rotation;

cipher = cipher.concat(String.fromCharCode(character));
```

此步骤后两个函数的代码如下：

```JavaScript
function cipher(text, shift) {
  let cipher = '';
  shift = shift % NUMBER_LETTERS;
  for (let i = 0; i < text.length; i++) {
    const isOutAlphabet =
      isOutLowerCharacterCipher(text, i, shift) ||
      isOutUpperCharacterCipher(text, i, shift);
    const rotation = isOutAlphabet ? NUMBER_LETTERS : NO_ROTATION;
    const character = text.charCodeAt(i) + shift - rotation;

    cipher = cipher.concat(String.fromCharCode(character));
  }
  return cipher.toString();
}

function decipher(text, shift) {
  let cipher = '';

  shift = shift % NUMBER_LETTERS;
  for (let i = 0; i < text.length; i++) {
    const isOutAlphabet =
      isOutLowerCharacterDecipher(text, i, shift) ||
      isOutUpperCharacterDecipher(text, i, shift);
    const rotation = isOutAlphabet ? NUMBER_LETTERS : NO_ROTATION;
    const character = text.charCodeAt(i) - shift + rotation;
    cipher = cipher.concat(String.fromCharCode(character));
  }
  return cipher.toString();
}
```

## Step 8. 变量命名

完成算法重构的最后一步是将循环中的变量 `i` 重命名为一个更合适的名称，如 `position`（这个更改可能看起来很“渺小”，但我们为变量指定语义值非常重要，包括循环中的经典 `i`、 `j` 和 `k`。

应用这些简单的步骤之后，我们算法的最终结果如下：

```JavaScript
function cipher(text, shift) {
  let cipher = '';
  shift = shift % NUMBER_LETTERS;
  for (let position = 0; position < text.length; position++) {
    const isOutAlphabet =
      isOutLowerCharacterCipher(text, position, shift) ||
      isOutUpperCharacterCipher(text, position, shift);
    const rotation = isOutAlphabet ? NUMBER_LETTERS : NO_ROTATION;
    const character = text.charCodeAt(position) + shift - rotation;

    cipher = cipher.concat(String.fromCharCode(character));
  }
  return cipher.toString();
}

function decipher(text, shift) {
  let cipher = '';

  shift = shift % NUMBER_LETTERS;
  for (let position = 0; position < text.length; position++) {
    const isOutAlphabet =
      isOutLowerCharacterDecipher(text, position, shift) ||
      isOutUpperCharacterDecipher(text, position, shift);
    const rotation = isOutAlphabet ? NUMBER_LETTERS : NO_ROTATION;
    const character = text.charCodeAt(position) - shift + rotation;
    cipher = cipher.concat(String.fromCharCode(character));
  }
  return cipher.toString();
}
```

## 总结

在这篇文章中，我们提出了一些建议，将最初的原始代码重构为可理解的代码。

在这篇文章中，通过一个案例逐步的演示了重构的过程。或许你觉得某些重构方式是不合适的，不可否认也存在其他重构方式。基于以上各种情况，你也可以将你的建设性想法分享给整个社区。

最后，我们本文主要讨论了以下几点：

- 魔法数字
- 从 `if-else` 中提取相似代码
- 避免使用 `else`
- 合并 IF 逻辑
- 简化算法逻辑
- 封装条件
- 移除 `if-else` 控制结构
- 变量命名

最后，我会把代码留给下，包括原始代码和最终代码，这样你就可以做最后的权衡了。

```JavaScript
function cipher(text, shift) {
  var cipher = '';
  shift = shift % 26;
  for (var i = 0; i < text.length; i++) {
    if (text.charCodeAt(i) >= 65 && text.charCodeAt(i) <= 90) {
      if (text.charCodeAt(i) + shift > 90) {
        cipher = cipher.concat(
          String.fromCharCode(text.charCodeAt(i) + shift - 26),
        );
      } else {
        cipher = cipher.concat(String.fromCharCode(text.charCodeAt(i) + shift));
      }
    } else if (text.charCodeAt(i) >= 97 && text.charCodeAt(i) <= 122) {
      if (text.charCodeAt(i) + shift > 122) {
        cipher = cipher.concat(
          String.fromCharCode(text.charCodeAt(i) + shift - 26),
        );
      } else {
        cipher = cipher.concat(String.fromCharCode(text.charCodeAt(i) + shift));
      }
    } else {
      // blank space
      cipher = cipher.concat(String.fromCharCode(text.charCodeAt(i) + shift));
    }
  }
  return cipher.toString();
}

function decipher(text, shift) {
  var decipher = '';
  shift = shift % 26;
  for (var i = 0; i < text.length; i++) {
    if (text.charCodeAt(i) >= 65 && text.charCodeAt(i) <= 90) {
      if (text.charCodeAt(i) - shift < 65) {
        decipher = decipher.concat(
          String.fromCharCode(text.charCodeAt(i) - shift + 26),
        );
      } else {
        decipher = decipher.concat(
          String.fromCharCode(text.charCodeAt(i) - shift),
        );
      }
    } else if (text.charCodeAt(i) >= 97 && text.charCodeAt(i) <= 122) {
      if (text.charCodeAt(i) - shift < 97) {
        decipher = decipher.concat(
          String.fromCharCode(text.charCodeAt(i) - shift + 26),
        );
      } else {
        decipher = decipher.concat(
          String.fromCharCode(text.charCodeAt(i) - shift),
        );
      }
    } else {
      // blank space
      decipher = decipher.concat(
        String.fromCharCode(text.charCodeAt(i) - shift),
      );
    }
  }
  return decipher.toString();
}

console.assert(
  cipher('Hello World', 1) === 'Ifmmp!Xpsme',
  `${cipher('Hello World', 1)} === 'Ifmmp!Xpsme'`,
);
console.assert(
  decipher(cipher('Hello World', 3), 3) === 'Hello World',
  `${decipher(cipher('Hello World', 3), 3)} === 'Hello World'`,
);
```

下面是最终的代码：

```JavaScript
const NUMBER_LETTERS = 26;
const NO_ROTATION = 0;
const LETTER = {
  a: 65,
  z: 90,
  A: 97,
  Z: 122,
};

function isOutLowerCharacterCipher(text, position, shift) {
  return (
    text.charCodeAt(position) >= LETTER.a &&
    text.charCodeAt(position) <= LETTER.z &&
    text.charCodeAt(position) + shift > LETTER.z
  );
}
function isOutUpperCharacterCipher(text, position, shift) {
  return (
    text.charCodeAt(position) >= LETTER.A &&
    text.charCodeAt(position) <= LETTER.Z &&
    text.charCodeAt(position) + shift > LETTER.Z
  );
}

function isOutLowerCharacterDecipher(text, position, shift) {
  return (
    text.charCodeAt(position) >= LETTER.a &&
    text.charCodeAt(position) <= LETTER.z &&
    text.charCodeAt(position) - shift < LETTER.a
  );
}

function isOutUpperCharacterDecipher(text, position, shift) {
  return (
    text.charCodeAt(position) >= LETTER.A &&
    text.charCodeAt(position) <= LETTER.Z &&
    text.charCodeAt(position) - shift < LETTER.A
  );
}

function cipher(text, shift) {
  let cipher = '';
  shift = shift % NUMBER_LETTERS;
  for (let position = 0; position < text.length; position++) {
    const isOutAlphabet =
      isOutLowerCharacterCipher(text, position, shift) ||
      isOutUpperCharacterCipher(text, position, shift);
    const rotation = isOutAlphabet ? NUMBER_LETTERS : NO_ROTATION;
    const character = text.charCodeAt(position) + shift - rotation;

    cipher = cipher.concat(String.fromCharCode(character));
  }
  return cipher.toString();
}

function decipher(text, shift) {
  let cipher = '';

  shift = shift % NUMBER_LETTERS;
  for (let position = 0; position < text.length; position++) {
    const isOutAlphabet =
      isOutLowerCharacterDecipher(text, position, shift) ||
      isOutUpperCharacterDecipher(text, position, shift);
    const rotation = isOutAlphabet ? NUMBER_LETTERS : NO_ROTATION;
    const character = text.charCodeAt(position) - shift + rotation;
    cipher = cipher.concat(String.fromCharCode(character));
  }
  return cipher.toString();
}

console.assert(
  cipher('Hello World', 1) === 'Ifmmp!Xpsme',
  `${cipher('Hello World', 1)} === 'Ifmmp!Xpsme'`,
);
console.assert(
  decipher(cipher('Hello World', 3), 3) === 'Hello World',
  `${decipher(cipher('Hello World', 3), 3)} === 'Hello World'`,
);
```
