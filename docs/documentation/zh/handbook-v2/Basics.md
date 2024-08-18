---
title: 基础
layout: docs
permalink: /zh/docs/handbook/2/basic-types.html
oneline: "学习 TypeScript 的第一步：基本类型。"
preamble: >
  <p>欢迎来到手册的第一页。如果这是您首次接触 TypeScript，不妨先从 '<a href='https://www.typescriptlang.org/docs/handbook/intro.html#get-started'>入门</a>' 指南开始。</p>
---

JavaScript 中的每个值会随着我们操作的不同表现出一系列的行为。
这听起来很抽象，但举个简单的例子，考虑一下我们可能在名为 `message` 的变量上运行的一些操作。

```js
// 访问 'message' 上的属性 'toLowerCase'
// 然后调用它
message.toLowerCase();

// 调用 'message'
message();
```

如果我们将这个过程分解，代码的第一行访问了一个叫做 toLowerCase 的属性，然后调用了它。
第二行尝试直接调用 message。

但假设我们不知道 message 的值，这是很常见的，我们就无法可靠地说我们尝试运行任何这样的代码会得到什么结果。
每个操作的行为完全取决于我们首先拥有的值是什么。

- `message` 可以被调用吗？
- 它是否有一个名叫 `toLowerCase` 的属性？
- 如果有，`toLowerCase` 是否可以被调用？
- 如果这两个值都可以被调用，它们会返回什么？

我们通常会在编写 JavaScript 时记住这些问题的答案，并希望我们能把所有的细节都记对。

例子中，假设 message 有如下定义：

```js
const message = "Hello World!";
```

你可能已经猜到，如果我们试图运行 message.toLowerCase()，我们会得到相同的字符串，只是字母全都变成小写。

那么第二行代码呢？
如果你熟悉 JavaScript，你会知道这会抛一个异常：

```txt
TypeError: message is not a function
```

如果我们能避免这种错误就太好了。

当我们运行代码时，我们的 JavaScript 运行时选择如何处理的方式是通过确定值的 _类型_——它具有什么样的行为和能力。
那就是 `TypeError` 所提示的内容 - 它说字符串 "Hello World!" 不能作为一个函数来调用。

对于某些值，如原始类型 `string` 和 `number`，我们可以使用 `typeof` 操作符在运行时识别它们的类型。
但对于其他的东西，比如函数，就没有相应的运行时机制来识别它们的类型。
例如，考虑这个函数： 

```js
function fn(x) {
  return x.flip();
}
```

我们可以通过阅读代码 _观察_，如果给这个函数一个带有可调用 flip 属性的对象，这个函数就能正常工作，但 JavaScript 并没有暴露出这个信息, 以我们可以在代码运行时检查的方式。
在纯 JavaScript 里唯一能知道 fn 会对特定值做什么的方式，就是调用它看看会发生什么。
这种行为使得在运行前预测代码将做什么变得困难，这意味着在编写代码时更难知道你的代码会做什么。

从这个角度看，_类型_ 指的是是一个概念，来描述哪些值能传给 fn ，哪些会导致程序崩溃。
JavaScript 实际上只提供了 _动态_ 类型 - 只能运行代码看看会发生什么。

另一种选择是使用 _静态_ 类型系统在代码运行 _之前_ 预测它会发生什么。

## 静态类型检查

回想一下我们早些时候因调用 `string` 类型的 `message` 而收到的 `TypeError`。
大部分人_都不喜欢_在运行他们的代码时碰到任何类型的错误 - 这些都被认为是 bug！
当我们编写新的代码时，我们会尽力避免引入新的 bug。

如果我们添加了一些代码，保存文件，重新运行代码，立刻发现有个错误，我们可能可以迅速地定位问题；但情况并非总是如此。
我们可能没有足够彻底地测试功能，所以我们可能测试不到一些潜在错误！
或者幸运的是，我们触发了这个错误，但我们可能已经进行了大规模的重构并添加了许多不同的代码，这就使我们不得不非常仔细的阅读大量代码来修复。

理想情况下，我们能有一个工具在代码运行 _之前_ 帮助我们发现这些 bug。
这就是 TypeScript 这样的静态类型检查器的作用。
_静态类型系统_描述了我们运行程序时值的结构和行为。
像 TypeScript 这样的类型检查器使用这些信息告诉我们事情何时开始偏离预期。

```ts twoslash
// @errors: 2349
const message = "hello!";

message();
```

上述 TypeScript 样例将在我们首次运行代码之前给我们一个错误提示。

## 非异常错误（Non-exception Failures）

到目前为止，我们讨论了一些事情，比如运行时错误 —— 这些情况是 JavaScript 运行时告诉我们它认为某些事情是无意义的。
这些情况出现是因为[ECMAScript 规范](https://tc39.github.io/ecma262/) 明确指定了当语言遇到意外的东西时，应该如何行事。

例如，规范规定，尝试调用不可调用的东西应该抛出一个错误。
也许，这听起来像是"显而易见的行为"，但你可能会想象访问一个对象上不存在的属性也应该抛出一个错误。
相反，JavaScript给我们的行为却不同，它返回的值是 `undefined`:

```js
const user = {
  name: "Daniel",
  age: 26,
};

user.location; // returns undefined
```

最后，静态类型系统会确定哪些代码应该标注为错误，这些问题可能在运行时不会立即显现为错误，甚至在表面上看似符合'有效'的JavaScript编码标准。
在 TypeScript 中，以下代码会产生一个关于 location 未定义的错误：

```ts twoslash
// @errors: 2339
const user = {
  name: "Daniel",
  age: 26,
};

user.location;
```

尽管有时候可能会在某些情况下牺牲代码的表达力，但是为了捕捉程序中那些隐蔽却真实的 bug, 
而且 TypeScript 捕获了 _许多_ 隐蔽 bug。

例如: 拼写错误,

```ts twoslash
// @noErrors
const announcement = "Hello World!";

// 您能多快发现这些拼写错误？ ?
announcement.toLocaleLowercase();
announcement.toLocalLowerCase();

// 我们可能原本想写这个......
announcement.toLocaleLowerCase();
```

未调用的函数,

```ts twoslash
// @noUnusedLocals
// @errors: 2365
function flipCoin() {
  // 原本应该是 Math.random()
  return Math.random < 0.5;
}
```

或者基本的逻辑错误。

```ts twoslash
// @errors: 2367
const value = Math.random() < 0.5 ? "a" : "b";
if (value !== "a") {
  // ...
} else if (value === "b") {
  // Oops, 不能达到
}
```

## 类型工具

TypeScript 可以捕获我们代码中的错误。
更好的是, TypeScript _还_ 可以防止我们犯这些错误。

类型检查器可以检查我们是否在变量和其他属性上访问了正确的属性。
一旦它具有了这种信息，它还可以开始 _建议_ 你可能想要使用的属性。

这意味着 TypeScript 也可以用于编辑代码，核心的类型检查器可以在你在编辑器中输入时提供错误消息和代码补全提示。
这是人们经常提到的 TypeScript 中的工具部分。

<!-- prettier-ignore -->
```ts twoslash
// @noErrors
// @esModuleInterop
import express from "express";
const app = express();

app.get("/", function (req, res) {
  res.sen
//       ^|
});

app.listen(3000);
```

TypeScript 在工具层面的作用非常强大，远不止拼写时进行代码补全和错误信息提示。
支持 TypeScript 的编辑器能够提供"快速修复"功能，自动修复错误，提供重构功能，方便对代码进行重组，以及实用的导航功能，便于跳转到变量的定义，或者查找给定变量的所有引用。
所有这些功能都是基于类型检查器构建的，并且完全跨平台，因此很有可能 [你最喜欢的编辑器已经提供了 TypeScript 的支持](https://github.com/Microsoft/TypeScript/wiki/TypeScript-Editor-Support).

## `tsc`, 即 TypeScript 编译器

我们已经谈到了类型检查，但我们还没有利用到我们的类型-_检查器_。
让我们来熟悉一下我们的新朋友 tsc，也就是 TypeScript 编译器。
首先，我们需要通过 npm 来获取它。

```sh
npm install -g typescript
```

> 这将全局安装 TypeScript 编译器 `tsc`。
> 如果你更倾向于从本地的 `node_modules` 包中运行 `tsc`，可以使用 `npx` 或者类似的工具。

现在，让我们移到一个空文件夹，尝试编写我们的第一个 TypeScript 程序：`hello.ts`。

```ts twoslash
// Greets the world.
console.log("Hello world!");
```

注意这里没有什么花哨的东西；这个"hello world"程序看起来和你在JavaScript中写的"hello world"程序一模一样。
现在我们来进行类型检查，运行 `typescript` 包为我们安装的 `tsc` 命令。

```sh
tsc hello.ts
```

好啦！

等等，"好啦"什么意思呢？
我们运行了 `tsc`，但什么也没发生！
其实，因为没有类型错误，所以我们的控制台没有任何输出，因为没有什么需要报告的。

但是再检查一遍 - 我们得到了一些 _文件_ 输出。
如果我们查看当前的目录，会发现在 `hello.ts` 旁边有一个 `hello.js` 文件。
这就是我们运行 `tsc` _编译_ 或者 _转换_ `hello.ts` 文件后得到的纯 JavaScript 文件输出。
如果我们查看文件内容，就能看到 TypeScript 在处理 `.ts` 文件后的输出结果：

```js
// Greets the world.
console.log("Hello world!");
```

在这个例子中，由于 TypeScript 需要转换的内容非常少，所以转换后的结果看起来与我们编写的内容完全一样。
编译器试图输出干净、可读的代码，看起来就像人写的代码一样。
虽然这并不总是那么容易，但 TypeScript 会保持一致的缩进，注意到我们的代码跨越了不同的代码行，试图保留周围的注释。

如果我们 _确实_ 引入了一个类型错误呢？
我们重新编写 `hello.ts`：

```ts twoslash
// @noErrors
// This is an industrial-grade general-purpose greeter function:
function greet(person, date) {
  console.log(`Hello ${person}, today is ${date}!`);
}

greet("Brendan");
```

如果我们再次运行 `tsc hello.ts`，注意到我们在命令行上得到了一个错误！

```txt
Expected 2 arguments, but got 1.
```

TypeScript 在告诉我们，我们忘记向 `greet` 函数传递一个参数，这是完全正确的。
到目前为止，我们只写了标准的 JavaScript，但类型检查仍然能够找到我们代码中的问题。
谢谢 TypeScript！

## 输出错误信息

你可能没有注意到，在上一个示例中，我们的 `hello.js` 文件再次发生了变化。
如果我们打开这个文件，会发现它的内容基本上还是和我们的输入文件差不多。鉴于 `tsc` 报告我们的代码存在错误，
这可能有些令人惊讶，但这是基于 TypeScript 的一个核心价值：大部分时间里，_你_ 会比 TypeScript 更了解情况。

重申之前的话，对代码进行类型检查将限制你可以运行的程序种类，因此，类型检查器能接受的事情类型是需要权衡的。
大多数时候这没问题，但也有一些情况下，这些检查会成为阻碍。
例如，设想你正在将 JavaScript 代码迁移到 TypeScript，并且引入了类型检查错误。
最终，你会处理这些类型检查上的问题，但是，原来的 JavaScript 代码已经可以正常工作了！
为什么将它转化为 TypeScript 后就不能运行呢？

所以 TypeScript 不会阻止你的开发进程。
当然，随着时间的推移，您可能希望更加防范错误，让 TypeScript 的检查更严格一些。

在这种情况下，你可以使用 [`noEmitOnError`](/tsconfig#noEmitOnError) 编译器选项。尝试修改你的 `hello.ts` 文件，并使用该参数运行 `tsc`：

```sh
tsc --noEmitOnError hello.ts
```

你会发现 `hello.js` 没有更新。

## 显式类型

到现在为止，我们还没有告诉 TypeScript `person` 或 `date` 是什么。
让我们修改代码，告诉 TypeScript `person` 是一个字符串, 而 `date` 应该是一个 `Date` 对象。
我们还会在 `date` 上使用 `toDateString()` 方法。

```ts twoslash
function greet(person: string, date: Date) {
  console.log(`Hello ${person}, today is ${date.toDateString()}!`);
}
```

我们做的是在 person 和 date 上添加了 类型注解 来描述 greet 可以被调用的值的类型。
你可以描述函数签名为：“greet 接收一个 string 类型的 person，和一个 Date 类型的 date”。

有了这个，TypeScript 就可以告诉我们其他可能误调用 greet 的情况。
例如: 

```ts twoslash
// @errors: 2345
function greet(person: string, date: Date) {
  console.log(`Hello ${person}, today is ${date.toDateString()}!`);
}

greet("Maddison", Date());
```

咦？
TypeScript 在我们的第二个参数上报告了一个错误，但为什么呢？

或许令人惊讶，在 JavaScript 中调用 `Date()` 实际上返回的是一个 `string`。
另一方面，通过 `new Date()` 构造一个 `Date` 实际上可以得到我们期望的结果。

无论如何，我们可以快速修复这个错误：

```ts twoslash {4}
function greet(person: string, date: Date) {
  console.log(`Hello ${person}, today is ${date.toDateString()}!`);
}

greet("Maddison", new Date());
```

请记住，我们并不总是需要写明确的类型注解。
在很多情况下，即使我们省略了它们，TypeScript 也可以 推断（或者说“弄清楚”）我们的类型。

```ts twoslash
let msg = "hello there!";
//  ^?
```

尽管我们没有告诉 TypeScript msg 的类型是 string，但它还是能够弄清楚。
这是一个特性，当类型系统最终会推断出相同的类型时，最好不要添加注解。

> 注意：在前面的代码示例中，消息气泡是你将鼠标悬停在单词上时编辑器会显示的内容。

## 擦除类型

让我们看一下，当我们用 tsc 编译上述的 greet 函数以生成 JavaScript 代码时，将会发生什么：

```ts twoslash
// @showEmit
// @target: es5
function greet(person: string, date: Date) {
  console.log(`Hello ${person}, today is ${date.toDateString()}!`);
}

greet("Maddison", new Date());
```

这里请注意两件事：

1. 我们的“person”和“date”参数不再有类型注释。
2. 我们的“模板字符串”——使用反引号（"`` "字符）的字符串——被转换为带有连接的纯字符串。

稍后我们会更深入地讨论第二点，但现在让我们专注于第一点。
类型注解并不属于 JavaScript（或者严格来说，不属于 ECMAScript），因此真的没有任何浏览器或其他运行时能够直接运行未经修改的 TypeScript。
这就是为什么 TypeScript 首先需要一个编译器 - 它需要某种方式来剔除或转换任何专属于 TypeScript 的代码，这样你才能运行它。
大多数专属于 TypeScript 的代码都会被擦除掉，所以我们的类型注解也被完全擦除了。

> 请记住：类型注解永远不会改变你程序的运行时行为。

## Downleveling

另一点不同是，我们的模板字符串被重写了，从

```js
`Hello ${person}, today is ${date.toDateString()}!`;
```

到

```js
"Hello ".concat(person, ", today is ").concat(date.toDateString(), "!");
```

为什么会发生这种情况？

模板字符串是 ECMAScript 2015 版本（又名 ECMAScript 6、ES2015、ES6 等等）的一项功能。
TypeScript 能够将代码从较新版本的 ECMAScript 重写为旧版本，例如 ECMAScript 3 或 ECMAScript 5（又名 ES3 和 ES5）。
从较新或“较高”版本的 ECMAScript 迁移到较旧或“较低”版本的过程有时称为 _降级_。

默认情况下，TypeScript 会转化为 ES3 代码，这是一个非常旧的 ECMAScript 版本。
我们可以使用 [`target`](https://www.typescriptlang.org/tsconfig#target) 选项将代码往较新的 ECMAScript 版本转换。
使用 `--target es2015` 参数运行会将 TypeScript 更改为目标 ECMAScript 2015，这意味着代码应该能够在支持 ECMAScript 2015 的任何地方运行。
因此，运行 `tsc --target es2015 hello.ts` 会给出以下输出：

```js
function greet(person, date) {
  console.log(`Hello ${person}, today is ${date.toDateString()}!`);
}
greet("Maddison", new Date());
```

> 尽管默认的目标是 ES3，但是大多数现代浏览器都已支持 ES2015。
> 因此，除非需要兼容某些非常老旧的浏览器，大部分开发者实际上可以安全地把 ES2015 或更高版本设为目标。

## Strictness

不同的用户使用 TypeScript 时对类型检查器寻求的特性也大不相同。
有些人更愿意选择更为松散的、可选择性的体验，这可以帮助他们验证程序的部分内容，同时还能享受良好的开发工具的辅助。
这就是 TypeScript 的默认体验，其中类型是可选的，推断总是选择最宽松的类型，而且并未对可能的 null/undefined 值进行检查。
就如同 tsc 在出现错误时仍会进行编译一样，这些默认设置是为了尽可能不影响你的工作。
如果你在将已有的 JavaScript 代码迁移到 TypeScript，这可能是一个理想的首次尝试方式。

相反，很多用户更希望 TypeScript 能够从一开始就进行尽可能多的验证，这也是语言提供严格性设置的原因。
这些严格性设置将静态类型检查从一个开关（你的代码不是被检查就是没有被检查）转变为更接近一个旋钮的设定。
你把这个旋钮拧得越紧，TypeScript 将会为你检查得越多。
这可能需要一些额外的工作，但通常来说，从长远来看，这是值得的，它能提供更彻底的检查和更准确的开发工具辅助。
只要可能，一个新的代码库应该总是打开这些严格性检查。

TypeScript 有几个可以开启或关闭的类型检查严格性标志，除非另有说明，否则我们的所有示例都将开启所有这些标志。
在命令行中的 [`strict`](/zh/tsconfig#strict) 标志，或在 [`tsconfig.json`](https://www.doc.tslang.org/zh/docs/handbook/tsconfig-json.html) 中使用 `"strict": true` 可以一次性全面启动这些选项，当然，我们也能选择单独地选出其中的某些进行关闭。
你应该了解的两个最重要的选项是 [`noImplicitAny`](/tsconfig#noImplicitAny) 和 [`strictNullChecks`](/tsconfig#strictNullChecks).

## `noImplicitAny`

回想一下，在某些地方，TypeScript 不会尝试为我们推断类型，而是回退到最宽松的类型：any。
这并不是最糟糕的事情 - 毕竟，回退到 any 本质上就是纯粹的 JavaScript 体验。

然而，经常使用 any 往往会破坏使用 TypeScript 的初衷。
你的程序类型越丰富，你将得到更多的验证和工具支持，意味着在编码过程中你会遇到更少的问题。
开启 [`noImplicitAny`](/zh/tsconfig#noImplicitAny) 标志会对任何隐式推断为 `any` 类型的变量发出错误警告。

## `strictNullChecks`

默认情况下，像 `null` 和 `undefined` 这样的值可以被赋值给任何其它类型的变量。
这可以使编写某些代码变得更容易，但是忘记处理 `null` 和 `undefined` 是世界上无数错误的原因－有些人认为这是一个[十亿美元的错误](https://www.youtube.com/watch?v=ybrQvs4x0Ps)！
[`strictNullChecks`](/zh/tsconfig#strictNullChecks) 选项使处理 `null` 和 `undefined` 更为明确，使我们 免于担心是否 _忘记_ 处理了 `null` 和 `undefined`。
