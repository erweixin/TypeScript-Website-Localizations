---
title: 给 JavaScript 程序员的 TypeScript 入门指南
short: 给 JavaScript 程序员的 TypeScript 入门指南
layout: docs
permalink: /zh/docs/handbook/typescript-in-5-minutes.html
oneline: 了解 TypeScript 如何扩展 JavaScript
---

TypeScript 与 JavaScript 有着一种特别的关系。TypeScript 提供了 JavaScript 所有的功能，并在此基础上增加了新的功能：TypeScript 的类型系统。

例如，JavaScript 提供了 string 和 number 这样的语言基础类型，但它不会检查你是否正确的使用这些类型。而 TypeScript 会进行这种检查。

这意味着你现有的 JavaScript 代码也是合法的 TypeScript 代码。TypeScript 的主要优点在于，它可以发现代码中的意外行为，从而降低 bug 出现的几率。

本教程简要概述了 TypeScript，重点介绍其类型系统。

## 通过推断确定类型

TypeScript 了解 JavaScript 语言，并且在许多情况下会为你生成类型。
例如，当创建一个变量并赋值给它时，TypeScript 会将根据值确定变量的类型。

```ts twoslash
let helloWorld = "Hello World";
//  ^?
```

通过理解 JavaScript 的工作原理，TypeScript 构建了一种类型系统，该系统能够接受 JavaScript 代码并添加类型。这种类型系统不需要在代码中额外添加字符来显式声明类型。就像上面的例子，TypeScript 知道 `helloWorld` 是一个 `string`。

你可能在 Visual Studio Code 中编写过 JavaScript，并且使用了编辑器的自动补全功能。Visual Studio Code 在背后使用 TypeScript 来使得编写 JavaScript 变得更加轻松。

## 定义类型

你可以在 JavaScript 中使用各种各样的设计模式。然而，有些设计模式使得类型的自动推断变得困难（例如，使用动态编程的模式）。为了覆盖这些情况，TypeScript 通过扩展 JavaScript 来确定类型。

例如，为了创建一个包含 `name: string` 和 `id: number` 的推断类型，你可以这样编写：

```ts twoslash
const user = {
  name: "Hayes",
  id: 0,
};
```

你可以使用 `interface` 声明来明确地描述这个对象的结构：

```ts twoslash
interface User {
  name: string;
  id: number;
}
```

然后，你可以通过在变量声明后使用 `:TypeName` 的语法来声明一个 JavaScript 对象符合你新的 `interface` ：

```ts twoslash
interface User {
  name: string;
  id: number;
}
// ---cut---
const user: User = {
  name: "Hayes",
  id: 0,
};
```

如果你提供的对象与你定义的 `interface` 不匹配，TypeScript会给你警告：

```ts twoslash
// @errors: 2322
interface User {
  name: string;
  id: number;
}

const user: User = {
  username: "Hayes",
  id: 0,
};
```

由于JavaScript支持类和面向对象编程，因此TypeScript也支持。你可以在类中使用 `interface` 声明：

```ts twoslash
interface User {
  name: string;
  id: number;
}

class UserAccount {
  name: string;
  id: number;

  constructor(name: string, id: number) {
    this.name = name;
    this.id = id;
  }
}

const user: User = new UserAccount("Murphy", 1);
```

你可以用 `interface` 来注解函数的参数和返回值：

```ts twoslash
// @noErrors
interface User {
  name: string;
  id: number;
}
// ---cut---
function deleteUser(user: User) {
  // ...
}

function getAdminUser(): User {
  //...
}
```

在 JavaScript 中，我们已经有了一小部分原始类型供使用：`boolean`, `bigint`, `null`, `number`, `string`, `symbol`, 以及 `undefined`，你可以在接口中使用它们。TypeScript 对这个列表进行了扩展，增加了一些新类型，例如 `any`（可以是任何类型），[`unknown`](/zh/play#example/unknown-and-never)（确保使用此类型的人明确声明了类型是什么），[`never`](/zh/play#example/unknown-and-never)（此类型永不可能发生），以及 `void`（一个返回 undefined 或者没有返回值的函数）。

你会发现有两种语法结构用来构建类型：[接口和类型](/zh/play/?e=83#example/types-vs-interfaces)。你应当优先考虑使用 `interface`。在你需要特定功能时，才用 `type`。

## 组合类型

使用 TypeScript，你可以通过组合简单的类型来创建复杂的类型。有两个常见的方法可以实现：联合类型（Unions）和泛型（Generics）。

### 联合类型（Unions）

通过联合类型，你可以声明一个类型可能是多种类型中的一种。例如，你可以描述 boolean 类型，它可以是 true 或 false：

```ts twoslash
type MyBool = true | false;
```

_注意：_如果你把鼠标悬停在上面的 MyBool 上，你会看到它被归类为 boolean。这是结构化类型系统的一个属性。下文会有更多的介绍。

联合类型的一个常见应用是描述一种值被允许的 `string` 或 `number` [字面量](/zh/docs/handbook/2/everyday-types.html#literal-types)集合：

```ts twoslash
type WindowStates = "open" | "closed" | "minimized";
type LockStates = "locked" | "unlocked";
type PositiveOddNumbersUnderTen = 1 | 3 | 5 | 7 | 9;
```

联合类型也提供了处理不同类型的方式。例如，你可能有一个函数，它接受一个 `array` 或者一个 `string`：

```ts twoslash
function getLength(obj: string | string[]) {
  return obj.length;
}
```

要了解变量的类型，可以使用 `typeof`：

| Type      | Predicate                          |
| --------- | ---------------------------------- |
| string    | `typeof s === "string"`            |
| number    | `typeof n === "number"`            |
| boolean   | `typeof b === "boolean"`           |
| undefined | `typeof undefined === "undefined"` |
| function  | `typeof f === "function"`          |
| array     | `Array.isArray(a)`                 |

例如，你可以编写一个函数，根据传入的是字符串还是数组返回不同的值：

<!-- prettier-ignore -->
```ts twoslash
function wrapInArray(obj: string | string[]) {
  if (typeof obj === "string") {
    return [obj];
//          ^?
  }
  return obj;
}
```

### 范性（Generics）

泛型为类型提供变量。一个常见的例子是数组。一个没有泛型的数组可以包含任何东西。一个带有泛型的数组可以描述数组所包含的值。

```ts
type StringArray = Array<string>;
type NumberArray = Array<number>;
type ObjectWithNameArray = Array<{ name: string }>;
```

你可以声明使用泛型的自定义类型：

```ts twoslash
// @errors: 2345
interface Backpack<Type> {
  add: (obj: Type) => void;
  get: () => Type;
}

// 这行代码告诉 TypeScript 存在一个名为 backpack 的常量，该变量的类型是 Backpack<string>

declare const backpack: Backpack<string>;

// object 是一个字符串，因为我们之前在 Backpack 中声明了 get 返回值的类型。
const object = backpack.get();

// 由于 backpack.add 的参数类型是字符串，所以你不能把一个数字传递给 add 函数。
backpack.add(23);
```

## 结构化类型系统

TypeScript 的核心原则之一就是类型检查注重值的 _"结构"_ 。这种方式有时被称为 "鸭子类型(duck typing)" 或 "结构化类型(structural typing)"。

在结构化类型系统中，如果两个对象有相同的结构，那么他们就被认为是同一类型。

```ts twoslash
interface Point {
  x: number;
  y: number;
}

function logPoint(p: Point) {
  console.log(`${p.x}, ${p.y}`);
}

// logs "12, 26"
const point = { x: 12, y: 26 };
logPoint(point);
```

`point` 变量从未被声明为 `Point` 类型。然而，在类型检查中，TypeScript 会比较 point 的结构和 Point 的结构。由于它们的结构相同，因此代码通过了。

只需要对象的部分字段匹配，这个结构匹配就能进行。

```ts twoslash
// @errors: 2345
interface Point {
  x: number;
  y: number;
}

function logPoint(p: Point) {
  console.log(`${p.x}, ${p.y}`);
}
// ---cut---
const point3 = { x: 12, y: 26, z: 89 };
logPoint(point3); // logs "12, 26"

const rect = { x: 33, y: 3, width: 30, height: 80 };
logPoint(rect); // logs "33, 3"

const color = { hex: "#187ABF" };
logPoint(color);
```

类和对象的结构匹配类似：

```ts twoslash
// @errors: 2345
interface Point {
  x: number;
  y: number;
}

function logPoint(p: Point) {
  console.log(`${p.x}, ${p.y}`);
}
// ---cut---
class VirtualPoint {
  x: number;
  y: number;

  constructor(x: number, y: number) {
    this.x = x;
    this.y = y;
  }
}

const newVPoint = new VirtualPoint(13, 56);
logPoint(newVPoint); // logs "13, 56"
```

如果对象或类拥有所有必需的属性，无论实现细节如何，TypeScript 都会认为它们匹配。

## 下一步

这是对日常 TypeScript 中使用的语法和工具的简要概述。参见：

- 阅读完整手册 [from start to finish](/zh/docs/handbook/intro.html)
- 探索 [Playground 示例](/zh/play#show-examples)
