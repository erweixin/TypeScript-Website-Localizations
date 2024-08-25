---
title: Typeof 类型操作符
layout: docs
permalink: /zh/docs/handbook/2/typeof-types.html
oneline: "在类型上下文中使用 typeof 操作符。"
---

## `typeof` 类型操作符

JavaScript 已经有一个 `typeof` 操作符，你可以在 _表达式_ 上下文中使用它：

```ts twoslash
// Prints "string"
console.log(typeof "Hello world");
```

TypeScript 增加了一个可以在类型上下文中使用的 `typeof` 操作符，用于引用变量或属性的 _类型_：

```ts twoslash
let s = "hello";
let n: typeof s;
//  ^?
```

对于基本类型来说，这并不是很有用，但是当与其他类型操作符结合使用时，`typeof` 可以方便地表达许多模式。
举个例子，让我们先看看预定义的类型 `ReturnType<T>`。
它接受一个 _函数类型_，并生成其返回类型：

```ts twoslash
type Predicate = (x: unknown) => boolean;
type K = ReturnType<Predicate>;
//   ^?
```

如果我们尝试对函数名使用 `ReturnType`，我们会看到一个有启发性的错误：

```ts twoslash
// @errors: 2749
function f() {
  return { x: 10, y: 3 };
}
type P = ReturnType<f>;
```

请记住，_值_ 和 _类型_ 不是同一回事。
要引用_值_ `f` 的 _类型_，我们使用 `typeof`：

```ts twoslash
function f() {
  return { x: 10, y: 3 };
}
type P = ReturnType<typeof f>;
//   ^?
```

### 限制

TypeScript 有意限制了你可以对哪些表达式使用 `typeof`。

具体来说，`typeof` 只能合法地用于标识符（即变量名）或它们的属性。
这有助于避免一个容易混淆的陷阱，即你以为正在执行的代码实际上并没有执行：

```ts twoslash
// @errors: 1005
declare const msgbox: (prompt: string) => boolean;
// type msgbox = any;
// ---cut---
// 本意是使用 = ReturnType<typeof msgbox>
let shouldContinue: typeof msgbox("Are you sure you want to continue?");
```
