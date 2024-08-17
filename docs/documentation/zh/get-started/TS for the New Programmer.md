---
title: 为新手程序员准备的 TypeScript 指南
short: 为新手程序员准备的 TypeScript 指南
layout: docs
permalink: /zh/docs/handbook/typescript-from-scratch.html
oneline: Learn TypeScript from scratch
---

恭喜你选择 TypeScript 作为你入门的编程语言之一——你已经做出了明智的选择！

你可能已经听说过，TypeScript 是 JavaScript 的一种“风格”或“变种”。
在现代编程语言中，TypeScript（TS）和 JavaScript（JS）之间的关系相当独特，了解这种关系将有助于你理解 TypeScript 是如何在 JavaScript 的基础上进行扩展的。

## 什么是 JavaScript? 简短的历史回顾

JavaScript（也称为 ECMAScript）最初作为一种简单的浏览器脚本语言诞生。
在其诞生之初，人们期望它用于嵌入网页中的短小代码片段，当时编写超过几十行代码都是比较罕见的。
因此，
早期的网页浏览器执行这些代码的速度相当慢。不过，随着时间的推移，JavaScript 变得越来越流行，网页开发者开始使用它来创建出丰富的互动内容。

网页浏览器开发者通过优化他们的执行引擎（动态编译）并扩展 JavaScript 的功能（增加了 API），来应对日益增长的 JavaScript 使用需求，这反过来又进一步提高了网页开发者对 JavaScript 的使用。
在现代网站上，你的浏览器经常运行着由数十万行代码构成的应用程序。
这是“网络”漫长而逐渐发展的过程，起初只是一个简单的静态网页网络，但现在已经发展成为支持各种丰富 _应用程序_ 的平台。

不仅如此，JavaScript 的使用已经流行到足以在浏览器上下文之外使用，如使用 node.js 实现 JavaScript 服务器。
JavaScript 具有 "无处不在" 的运行特性，这使得它成为跨平台开发的理想选择。
如今有很多开发者 _仅_ 使用 JavaScript 来编写他们的整个技术栈！

总的来说，我们有一种语言，它最初是为了解决一些快速使用的需求而设计的，但后来它发展成了一种全面的工具，可以用来编写拥有数百万行代码的应用程序。
每种语言都有自己的 _怪癖_ —— 奇怪和令人惊讶的地方，而 JavaScript 的朴素起源让它有 _很多_ 这样的地方。以下是一些例子：

- JavaScript 的等号运算符（==）会对其操作数进行 _类型转换_，这可能导致意料之外的行为：

  ```js
  if ("" == 0) {
    // 这是真的！但为什么呢？
  }
  if (1 < x < 3) {
    // 对于 *任何* x 的值，这都是 True！
  }
  ```

- JavaScript 还允许访问不存在的属性：

  ```js
  const obj = { width: 10, height: 15 };
  // 为什么这是 NaN？拼写是非常容易出错的！
  const area = obj.width * obj.heigth;
  ```

大多数编程语言在这类错误发生时都会抛出错误，有些甚至会在编译期 —— 也就是任何代码运行之前就进行错误提示。
在编写小规模的程序时，这些怪癖虽然令人恼火，但还是可以处理的；但是在编写含有数百或数千行代码的应用程序时，这些持续不断的意外就会成为一个严重的问题。

## TypeScript：静态类型检查器

我们之前提到，有些语言根本不会让那些有 bug 的程序运行。
在不运行代码的情况下检测错误被称为 _静态检查_。
根据被操作值的类型来判断什么是错误什么不是错误，这被称为静态 _类型_ 检查。

TypeScript 在执行程序之前对其进行错误检查，并根据 _值的类型_ 进行检查，使其成为一个 _静态类型检查器_。
例如，上述最后一个示例中的错误是由于 obj 的 _类型_ 所导致的。
下面是 TypeScript 找到的错误：

```ts twoslash
// @errors: 2551
const obj = { width: 10, height: 15 };
const area = obj.width * obj.heigth;
```

### JavaScript 的类型超集

那么，TypeScript 和 JavaScript 之间又是什么关系呢？

#### 语法

TypeScript 是一种 JavaScript 的超集语言: 所以，JS 的语法在 TS 中也是合法的。
语法指的是我们编写文本以形成程序的方式。
例如，因为缺少 `)`，这段代码存在一个 _语法_ 错误：

```ts twoslash
// @errors: 1005
let a = (4
```

TypeScript 不会因为语法问题将任何 JavaScript 代码视为错误。
也就是说，你可以将任何正常运行的 JavaScript 代码放入一个 TypeScript 文件中，而不必担心其具体书写方式。

#### 类型

然而，TypeScript 是一种带有类型的超集，这意味着它增加了关于如何使用不同类型值的规则。
之前关于 `obj.heigth` 的错误并不是 _语法_ 错误：而是以错误的方式引用了不存在的值。

再举一个例子，这是可以在浏览器中运行的 JavaScript 代码，并且它 _会_  log 一个值：

```js
console.log(4 / []);
```

这个语法合法的程序会 log `Infinity`。
不过，TypeScript 认为用数组进行除法运算是不合逻辑的操作，所以会报错：

```ts twoslash
// @errors: 2363
console.log(4 / []);
```

有可能你 _确实_ 想要将一个数字除以一个数组，或许只是为了看看会发生什么，但大多数情况下，这是一种编程错误。
TypeScript 的类型检查器旨在允许正确的程序通过，同时尽可能检测出常见的错误。
（稍后，我们会学习一些设置，用来配置 TypeScript 检查代码的严格程度。）

如果你将一些代码从 JavaScript 文件移动到 TypeScript 文件，你可能会看到一些 _类型错误_，这取决于代码的写法。
这些可能是代码中真正存在的问题，也可能是 TypeScript 过于保守。
在整个指南中，我们将演示如何添加各种 TypeScript 语法来消除这些错误。

#### 运行时行为

TypeScript 是保留了 JavaScript 的 _运行时行为_ 的一种编程语言。
例如，在 JavaScript 中，用零做除数会得到 `Infinity`，而不会抛出运行时异常。
**永远不会** 改变 JavaScript 代码的运行时行为是 Typescript 的原则。

这意味着，如果你将代码从 JavaScript 转移到 TypeScript，即使 TypeScript认为代码存在类型错误，也 **保证** 能以相同的方式运行。

保持与 JavaScript 相同的运行时行为是 TypeScript 的一个基础承诺，因为这意味着你可以轻松地在两种语言之间转换，而不用担心那些可能导致程序停止运行的细微差别。

<!--
Missing subsection on the fact that TS extends JS to add syntax for type
specification.  (Since the immediately preceding text was raving about
how JS code can be used in TS.)
-->

#### 移除类型

大致来说，TypeScript 编译器在检查完你的代码后，会 _移除类型_，以生成最终的“编译”代码。
也就是说，一旦你的代码编译完成，生成的纯 JavaScript 代码中是没有任何类型信息的。

这也意味着 TypeScript 绝不会根据它推断出的类型改变你程序的行为。
总的来说，虽然你可能在编译过程中看到类型错误，但类型系统本身对程序运行时的行为没有任何影响。

最后，TypeScript 不提供任何额外的运行时库。你的程序将使用与 JavaScript 程序相同的标准库或外部库，因此你不需要学习任何额外的 TypeScript 专用框架。

<!--
Should extend this paragraph to say that there's an exception of
allowing you to use newer JS features and transpile the code to an older
JS, and this might add small stubs of functionality when needed.  (Maybe
with an example --- something like `?.` would be good in showing readers
that this document is maintained.)
-->

## 学习 JavaScript 和 TypeScript

我们经常会看到这样的问题：“我应该学习 JavaScript 还是 TypeScript？”

答案是：你不能在不学习 JavaScript 的情况下学习 TypeScript！
TypeScript 与 JavaScript 共享语法和运行时行为，因此你学习 JavaScript 的任何内容同时也在帮助你学习 TypeScript。

有很多资源可以帮助程序员学习 JavaScript；如果你编写 TypeScript 代码，不应该忽视这些资源。
例如，StackOverflow 上带有 `javascript` 标签的问题约是带有 `typescript` 标签问题的 20 倍，但 _所有_ `javascript` 问题在 TypeScript 中同样适用。

如果你在搜索类似 “如何在 TypeScript 中排序一个列表” 的问题时，请记住：TypeScript 是 JavaScript 的运行时加上一个编译时类型检查器。
在 TypeScript 中排序列表的方法与你在 JavaScript 中使用的方法是一样的。如果你找到直接使用 TypeScript 的资源，那当然很好，但解决日常问题时，不要局限地认为你需要特定于 TypeScript 的答案。

## 下一步

以上是对日常使用 TypeScript 相关语法和工具的简要概述。从这里开始，你可以：

- 学习一些 JavaScript 的基础知识，我们推荐以下几种方式：

  - [微软的 JavaScript 资源](https://developer.microsoft.com/javascript/) or
  - [Mozilla Web Docs 上的 JavaScript 指南](https://developer.mozilla.org/docs/Web/JavaScript/Guide)

- 接下来查看 [针对 JavaScript 程序员的 TypeScript 教程](/zh/docs/handbook/typescript-in-5-minutes.html)
- 阅读完整手册 [from start to finish](/zh/docs/handbook/intro.html)
- 探索 [Playground 示例](/play#show-examples)

<!-- Note: I'll be happy to write the following... -->
<!--
## Types

    * What's a type? (For newbies)
      * A type is a *kind* of value
      * Types implicitly define what operations make sense on them
      * Lots of different kinds, not just primitives
      * We can make descriptions for all kinds of values
      * The `any` type -- a quick description, what it is, and why it's bad
    * Inference 101
      * Examples
      * TypeScript can figure out types most of the time
      * Two places we'll ask you what the type is: Function boundaries, and later-initialized values
    * Co-learning JavaScript
      * You can+should read existing JS resources
      * Just paste it in and see what happens
      * Consider turning off 'strict' -->
