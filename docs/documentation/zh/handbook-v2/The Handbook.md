---
title: TypeScript 手册
layout: docs
permalink: /zh/docs/handbook/intro.html
oneline: 学习 TypeScript 的第一步
handbook: "true"
---

## 关于本手册

在向编程社区推出20多年后，JavaScript 现在已经成为有史以来最广泛使用的跨平台语言之一。起初，它只是一种用于为网页添加简单交互性的小型脚本语言，如今已发展成为各种规模的前端和后端应用程序的首选语言。尽管用 JavaScript 编写的程序在规模、范围和复杂性上呈指数级增长，但 JavaScript 语言表达不同代码单元之间关系的能力并没有随之增强。再加上 JavaScript 颇为奇特的运行时语义，语言和程序复杂性之间的这种不匹配使得 JavaScript 开发在大规模管理上变得困难重重。

程序员编写的最常见的错误类型可以被描述为类型错误：在需要某种类型的值的地方使用了其他类型的值。这可能是由于简单的拼写错误、对库的API接口理解不足、对运行时行为的错误假设或其他错误导致的。TypeScript 的目标是成为 JavaScript 程序的静态类型检查器——换句话说，它是一个在您的代码运行之前运行（静态）并确保程序的类型正确（类型检查）的工具。

如果您在没有 JavaScript 背景的情况下接触 TypeScript，并打算将 TypeScript 作为您的第一门语言，我们建议您首先阅读[Microsoft Learn JavaScript tutorial](https://developer.microsoft.com/javascript/)或者[JavaScript at the Mozilla Web Docs](https://developer.mozilla.org/docs/Web/JavaScript/Guide)这两份教程。
如果您有其他语言的经验，通过阅读手册，您应该能够很快掌握 JavaScript 语法。

## 本手册结构

这本手册分为两个部分：

- **手册**

  《TypeScript 手册》旨在成为一份向日常程序员解释 TypeScript 的综合性文档。您可以通过左侧导航栏自上而下的顺序阅读该手册。

  每一章或每一页都能让您对给定的概念有深入的理解。虽然 TypeScript 手册不是完整的语言规范，但它旨在成为关于该语言的所有特性和行为的综合指南。

  完成手册导览的读者应该能够：

  - 阅读并理解常用的 TypeScript 语法和模式
  - 理解编译器重要选项的作用
  - 在大多数情况下正确预测类型系统的行为

  为了清晰和简洁起见，手册的主要内容不会探讨所涵盖特性的所有边缘 case 或细节。您可以在参考文章中找到关于特定概念的更多细节。

- **参考文件**

  手册导航栏下方的参考部分旨在让您更深入地了解 TypeScript 某一特定部分的工作原理。您可以自上而下阅读，但每个部分的目的都是对单个概念进行更深入的解释——这意味着不存在连续性的目标。

### 非目标事项

该手册还旨在成为一份简洁的文档，可以在几个小时内轻松阅读。为了简短起见，某些主题不会被涵盖。

具体来说，手册不会全面介绍像函数、类和闭包这样的核心 JavaScript 基础知识。在适当的地方，我们会提供一些背景知识的链接，您可以利用这些链接来了解那些概念。


手册也并不是用来替代语言规范的。在某些情况下，我们会跳过边缘情况或行为的形式描述，转而提供更高层次、易于理解的解释。对于 TypeScript 行为的许多方面，我们有单独的参考页面进行更精确和正式的描述。这些参考页面并不适用于不熟悉 TypeScript 的读者，所以它们可能会使用高级术语或提到您还未阅读过的主题。

最后，手册不会覆盖 TypeScript 与其他工具的交互，除非是必要的情况。像是如何将 TypeScript 配置成与 webpack、rollup、parcel、react、babel、closure、lerna、rush、bazel、preact、vue、angular、svelte、jquery、yarn 或 npm 结合使用，这些主题不在手册的范围之内 —— 您可以在网络上的其他地方找到相关资源。

## 开始

在开始阅读[基础知识](/zh/docs/handbook/2/basic-types.html)之前，我们建议先阅读以下其中一个入门页面。这些简介旨在突出 TypeScript 与您喜欢的编程语言之间的关键相似点和差异点，并澄清这些语言特有的常见误解。

- [给新程序员的 TypeScript 入门指南](/zh/docs/handbook/typescript-from-scratch.html)
- [给 JavaScript 程序员的 TypeScript 入门指南](/docs/handbook/typescript-in-5-minutes.html)
- [给 Java/C# 程序员的 TypeScript 入门指南](/zh/docs/handbook/typescript-in-5-minutes-oop.html)
- [给函数式编程程序员的 TypeScript 入门指南](/zh/docs/handbook/typescript-in-5-minutes-func.html)

否则，请跳转到[基础知识](/zh/docs/handbook/2/basic-types.html).
