---
title: JavaScript 代码整洁之道-概述篇
date: 2021-05-17 12:16:11
tags: JavaScript
---

> 本文源于翻译 [Clean Code Applied to JavaScript — Part I. Before Your Start](https://dev.to/carlillo/clean-code-applied-to-javascript-part-i-before-your-start-16ic)，点击底部的 “阅读原文”，可查看原文。

# JavaScript 代码整洁之道-概述篇

这篇文章是这一系列文章中的第一篇，它将对 "如何编写 JavaScript 整洁代码" 这个经常被提及的话题进行深入的探讨。

在本系列中，我们将介绍和讨论书写整洁代码的技巧，每个程序员都应该了解并掌握这些技巧，并将这些技巧应用于 JavaScript/TypeScript 语言。


## 简介

首先要理解一些关于编写整洁代码的概念。

1. 代码坏味道和重构

> 代码的坏味道是一种表象，它通常对应于系统中更深层次的问题 - Martin Fowler

> 代码的坏味道是导致技术债的因素 - Robert C. Martin

在我看来，Martin Fowler 和 Robert C. Martin 的说法是相互兼容的，Fowler 认为代码坏味道是导致系统问题的导火索，而 Martin 则指出代码坏味道会引起副作用。

2. 技术债

技术债是软件开发中的一个概念，开发人员为了加速软件开发，在应该采用最佳方案时进行了妥协，改用了短期内能加速软件开发的方案，从而在未来给自己带来的额外开发负担。

因此，就像生活本身一样，最理想的情况是不要有负债，要有一个健康的财务状况（有经验的程序员和完善的基础设施，可以让开发过程不会产生负面影响）。然而，在现实生活中，有时我们需要用贷款去大学学习或购买我们的第一辆汽车，我们会背上一些可以承担的债务，然后需要支付一点点利息。在软件开发中的也存在类似的情况，我们必须在日后偿还以前遗留下来的技术债。在没有存款和就业的情况下，没有人会去买一栋几百万的房子，无法偿还的债务会导致个人破产。同理无法偿还的技术债会导致软件开发失败。

**代码重构** 是在不改变其外部行为的情况下对现有项目代码进行重组的过程。

- 代码重构改善了软件的非功能属性
- 优点包括提升代码的可读性和降低复杂性
- 这些可以提高源代码的可维护性
- 创建一个更清晰的结构，以提高可扩展性

## 在开始之前

在开始介绍用 JavaScript 编写整洁代码的例子之前，不得不提一些对团队工作至关重要的建议。

## 代码可读性

代码必须是对人类是可读的。不要考虑计算机如何处理它，因为会有许多工具来转换我们的代码（编译器）。因此，最重要的是，代码将是人类可读的，因为你在开发代码时，最长的工作就是阅读代码，而不是写代码。

下面的例子中分别定义了一个存储用户的数组。这三个例子中，哪个更具有可读性？哪一个需要你付出更少的精力来进行阅读？那这个就是你应该组织代码的方式。


```JavaScript
    const users = [{ id: 1, name: "Carlos Caballero", memberSince: "1997–04–20", favoriteLanguageProgramming: ["JavaScript", "C", "Java"] }, { id: 2, name: "Antonio Villena", memberSince: "2014–08–15", favoriteLanguageProgramming: ["Go", "Python", "JavaScript"] }, { id: 3, name: "Jesús Segado", memberSice: "2015–03–15", favoriteLanguageProgramming: ["PHP", "JAVA", "JavaScript"] } ];

    /***********************/

    const users = [
    { id: 1, name: "Carlos Caballero", memberSince: "1997–04–20", favoriteLanguageProgramming: ["JavaScript", "C", "Java"] },
    { id: 2, name: "Antonio Villena", memberSince: "2014–08–15", favoriteLanguageProgramming: ["Go", "Python", "JavaScript"] },
    { id: 3, name: "Jesús Segado", memberSice: "2015–03–15", favoriteLanguageProgramming: ["PHP", "JAVA", "JavaScript"] },
    ];


    /***********************/

    const users = [{
     id: 1,
     name: "Carlos Caballero",
     memberSince: "1997–04–20",
     favoriteLanguageProgramming: ["JavaScript", "C", "Java"],
    },
    {
     id: 2,
     name: "Antonio Villena",
     memberSince: "2014–08–15",
     favoriteLanguageProgramming: ["Go", "Python", "JavaScript"],
    },
    {
     id: 3,
     name: "Jesús Segado",
     memberSince: "2015–03–15",
     favoriteLanguageProgramming: ["PHP", "JAVA", "JavaScript"],
    }];
```

## 使用英语编写代码

首先，我本人并不是一个能够熟练讲英语的人。我在这个行业遇到很大的困难就是，与母语相比，我几乎不能用英语进行良好和有趣的对话。但是在课堂上，我让我的学生们必须用英语写代码，我所有的代码也都是用英语编写的。即使是使用了糟糕的英语也远远比使用母语更好，除非你非常幸运，你的母语本身就是英语。这样做的原因是，英语目前被应用于全球的商业活动。你可能喜欢或不喜欢，但世界上每个人都明白，英语是与另一个国家交流时使用的语言，正如我之前告诉你的，你的大部分时间都在阅读代码。想象一下，阅读代码一种你没有接触的语言书写的代码，你将无法知道变量或函数的名称，所有的代码对你来说都是加密的。

因此，你必须用英语开发，即使它并不是你的母语。我们需要在工作中去学习英语。虽然英语不是母语，但我每天都在阅读和书写英语。当然有偶尔会犯错，但我们都能相互理解，因为在使用一种语言时，最重要的是将意思传达出来。

试着推断一下下面的代码片断是做什么的。也就是说，下面的代码片断是用不同的语言和英语编写的（显然，如果其中一种样本语言是你的母语，就能快速理解）。如果对下面的语言很熟悉，那可以用谷歌翻译成你不了解的语言，并尝试推导出代码的作用。

```JavaScript
 const benutzer = {
     id: 1,
     name: "John Smith",
     mitgliedVon: "1997–04–20",
    };

    Gehaltserhöhung(benutzer, 1000); 

    /***********************/

    const użytkownik = {
     id: 1,
     imię: "John Smith",
     członekZ: "1997–04–20",
    };
    wzrostWynagrodzeń(użytkownik, 1000);

    /***********************/

    const user = {
     id: 1,
     name: "John Smith",
     memberSince: "1997–04–20",
    };
    increaseSalary(user, 1000);
```

## 团队协作

很久以前，有一个程序员在开发一个软件项目... 曾经，这是一个美好的故事! 在我们开始学习软件开发的过程中，的确是一个人独自面对电脑书写代码去解决问题。但如今，一个人开发一个软件的情况已经不存在。

因此，我们必须学会如何在一个团队中工作。以下几点通常会引起争议：

- 用空格还是 Tab 来格式化代码
- 在函数的名称旁边还是在下一行写大括号
- 是否在语句的结尾处放一个分号

这些讨论你听得懂吗？对不起，这些讨论是荒谬的。因为所有团队都会有统一的标准，最后由代码格式化工具将代码修改为一致的风格。

当一个程序员打开一个项目文件并开始阅读代码的时候，他无法推断出这些代码是由他还是由同事编写的。这时不要慌，我们有强大的版本管理工具，如 GIT。

因此，要直接在团队中工作，我们需要：

不用指定特定的 IDE 开发，可以通过配置一个标准的 `.editorconfig` 文件，让团队各个成员使用各自喜欢的 IDE。并不是所有人都使用 WebStorm、VSCode 或 Eclipse，`.editorconfig` 帮助开发者在不同的编辑器 和 IDE 之间定义并保持一致的编码风格。

```JavaScript
 root = true

    [*]
    end_of_line = lf
    insert_final_newline = true

    [*.{js,py}]
    charset = utf-8

    [*.py]
    indent_style = space
    indent_size = 4

    [Makefile]
    indent_style = tab

    [*.js]
    indent_style = space
    indent_size = 2

    [{package.json,.travis.yml}]
    indent_style = space
    indent_size = 2
```

Lint 允许我们制定一些规则用于检查代码中的格式错误，每种语言都有自己的 Lint，必须要做在项目里面配置和使用它。不过有时也会有一些项目，其中的代码没有按照你喜欢的方式配置代码的规则，不过你仍然可以用这种方式进行编码，最终通过 IDE 格式化代码，从而节省一些时间。

```JavaScript
 {
     "globals": {
     "myModule": true
     },
     "env": {
     "browser": true
     },
     "rules": {
     "no-unused-vars": "error",
     "quotes": ["warning", "double"]
     }
    }
```

```JavaScript
    const a = "a" ;
    const b = a;
    const c = b;
```

有一个在业界广泛使用的工具被称为 Prettier，它能够根据 linter 的规则实时改变我们的代码格式（IDE的插件）。这使我们能够集中精力解决我们必须解决的问题，此外，我们通过成为一个团结的团队来生成干净的代码。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/27ac1559397b4004808c42f2aa91a327~tplv-k3u1fbpfcp-zoom-1.image)

## 结论

在这篇文章中，总结了在做整洁代码之前要关注的几个基本要点。这些要点对任何一个开发者都适用。

- **书写能让人读懂的代码。** 由于我们会花费大量的时间和精力去阅读代码，可读性良好的代码会让我们更容易理解。

- **使用英语编写代码。** 因为它目前是国际语言，只有这样我们才能与世界上的任何人分享代码。

- **团队协作。** 制定通用的规则，依靠工具让团队的代码风格保持统一，要让整个项目的代码看起来是由一个人编写的，消除个人的代码风格。

---

> 
> 译者注：
> 
> 本文是整个系列的第一篇，被放在了最后翻译的，但不影响文章的阅读体验，至此全部文章翻译完毕。回顾这一系列文章中提到的技巧和建议，几乎都可以在 《代码整洁之道》（作者 Robert C. Martin）这本书中找到。为了翻译得更准确，在翻译文章的过程中也参考了这本书上的内容。原文作者 [Carlos Caballero](https://github.com/Caballerog) 是一位计算机的 PhD，在西班牙工作和生活。正如他所说的那样，代码整洁的原则并不限于 JavaScript 语言，适用于各种开发语言。
> 
> 关于写代码这件事情，有的人说写过的代码像诗一样优雅，有的人吐槽祖传下来的代码像一座屎山。很多的时候，大家不是不想写出优雅的代码，而是不知道如何写出优雅的代码。在代码的 Code Review 中，经常看到一些奇奇怪怪的代码，这些代码是幼稚的、冗赘的。因此，学习和掌握整洁代码的原则和技巧是必要的，这也是翻译和学习这一系列文章最根本的目的。
