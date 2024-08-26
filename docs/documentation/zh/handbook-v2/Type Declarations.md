---
title: 类型声明
layout: docs
permalink: /zh/docs/handbook/2/type-declarations.html
oneline: "TypeScript 对无类型 JavaScript 的类型支持方式。"
---

在你已经阅读过的章节中，我们一直在使用所有 JavaScript 运行时中存在的内置函数来演示基本的 TypeScript 概念。
然而，如今几乎所有的 JavaScript 项目都包含许多库来完成常见任务。
为你应用程序中不属于你自己代码的部分提供类型，将大大改善你的TypeScript使用体验。
这些类型是从哪里来的呢？

## 类型声明是什么样子的？

假设你写了这样的代码：

```ts twoslash
// @errors: 2339
const k = Math.max(5, 6);
const j = Math.mix(7, 8);
```

TypeScript是如何知道 `max` 存在而 `mix` 不存在的，即使 Math 的实现并不是你代码的一部分？

答案是有描述这些内置对象的 _声明文件_。
声明文件提供了一种方式来 _声明_ 某些类型或值的存在，而无需为这些值提供实际的实现。

## `.d.ts` 文件

TypeScript主要有两种文件。
`.ts `文件是 _实现_ 文件，包含类型和可执行代码。
这些文件产生 `.js` 输出，通常是你写代码的地方。

`.d.ts` 文件是 _声明_ 文件，只包含类型信息。
这些文件不产生 `.js` 输出；它们只用于类型检查。
我们稍后将学习如何编写我们自己的声明文件。

## 内置类型声明

TypeScript 包含了 JavaScript 运行时中所有标准化内置 API 的声明文件。
这包括内置类型如 `string` 或 `function` 的方法和属性，顶级名称如 `Math` 和 `Object`，以及它们相关的类型。
默认情况下，TypeScript 还包括在浏览器中运行时可用的类型，如 `window` 和 `document`；这些统称为DOM API。

TypeScript 使用 `lib.[something].d.ts` 的模式来命名这些声明文件。
如果你导航到这样命名的文件，你就知道你正在处理平台的某个内置部分，而不是用户代码。

### `target` 设置

实际上，你可以使用的方法、属性和函数会根据你的代码运行的 JavaScript 版本而变化。
例如，字符串的 `startsWith` 方法只从被称为 _ECMAScript 6_ 的 JavaScript 版本开始可用。

了解你的代码最终运行在哪个JavaScript版本上很重要，因为你不想使用比你部署的平台更新(new version)的版本中的 API。
这是 [`target`](/zh/tsconfig#target) 编译器设置的一个功能。

TypeScript 通过根据你的 [`target`](/zh/tsconfig#target) 设置来改变默认包含的 `lib` 文件来帮助解决这个问题。
例如，如果 [`target`](/zh/tsconfig#target) 是 ES5，当你尝试使用 startsWith 方法时，你会看到一个错误，因为该方法只在 `ES6` 或更高版本中可用。

### `lib` 设置

[`lib`](/zh/tsconfig#lib) 设置允许更精细地控制在你的程序中哪些内置声明文件被认为是可用的。
有关更多信息，请参见 [`lib`](/zh/tsconfig#lib) 的文档页面。

## 外部定义

对于非内置API，有多种方式可以获取声明文件。
具体如何做取决于你要获取类型的库。

### 捆绑类型

如果你使用的库是作为npm包发布的，它可能已经在其分发包中包含了类型声明文件。
你可以阅读项目的文档来了解，或者简单地尝试导入包，看看TypeScript是否能自动解析类型。

如果你是一个包作者，考虑将类型定义与你的包捆绑在一起，你可以阅读我们关于 [捆绑类型定义]((/docs/handbook/declaration-files/publishing.html#including-declarations-in-your-npm-package)) 的指南。

### DefinitelyTyped / `@types`

[DefinitelyTyped 仓库](https://github.com/DefinitelyTyped/DefinitelyTyped/) 是一个集中存储数千个库的声明文件的中央仓库。
绝大多数常用库都在DefinitelyTyped上有可用的声明文件。

DefinitelyTyped上 的定义也会自动发布到 npm 上，使用 `@types` 作用域。
类型包的名称总是与底层包本身的名称相同。
例如，如果你安装了 `react` npm包，你可以通过运行以下命令安装其对应的类型：

```sh
npm install --save-dev @types/react
```

TypeScript会自动在 `node_modules/@types` 下查找类型定义，所以无需额外步骤就可以在你的程序中使用这些类型。

### 你自己的定义

在罕见的情况下，如果一个库既没有捆绑自己的类型，也没有在 DefinitelyTyped 上有定义，你可以自己编写声明文件。
请参阅附录 [编写声明文件](/zh/docs/handbook/declaration-files/introduction.html) 以获取指南。

如果你想在不编写声明文件的情况下消除对特定模块的警告，你也可以通过在项目的 `.d.ts` 文件中为其放置一个空声明，将该模块快速声明为 any 类型。
例如，如果你想使用一个名为 `some-untyped-module` 的模块，但没有它的定义，你可以这样写：

```ts twoslash
declare module "some-untyped-module";
```
