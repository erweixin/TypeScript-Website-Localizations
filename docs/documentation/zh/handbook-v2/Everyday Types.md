---
title: 日常类型
layout: docs
permalink: /zh/docs/handbook/2/everyday-types.html
oneline: "typeScript 中的原始类型"
---

在本章节中，我们会介绍您在 JavaScript 代码中可能遇到的一些最常见的值类型，以及在 TypeScript 中描述这些类型的对应方式。
这并不是一个详尽无遗的列表，未来的章节将描述更多的类型及其命名和使用方式。

类型也可以出现在许多除了类型注释之外的 _地方_。
当我们学习这些类型本身时，也会了解到在哪些地方我们可以引用这些类型来构建新的结构。

我们将从回顾您在编写 JavaScript 或 TypeScript 代码时可能遇到的最基本和常见的类型开始。
这些类型将在后续成为更复杂类型的核心构建模块。

## 原始类型：`string`，`number`，和 `boolean`

JavaScript有三种非常常见的[原始类型](https://developer.mozilla.org/en-US/docs/Glossary/Primitive): `string`, `number`, and `boolean`.
每一种都在TypeScript中有对应的类型。
正如您可能预料的那样，这些都是在对这些类型的值使用 JavaScript 的 `typeof` 操作符时会看到的名称：

- `string`  表示像 `Hello, world` 这样的字符串值
- `number` 是表示像 `42` 这样的数字。JavaScript 没有为整数设置一个特殊的运行时值，因此没有等同于 `int` 或 `float` 的东西 - 一切都是 `number`
- `boolean` 用于 `true` 和 `false` 这两个值

> 类型名称 `String`, `Number`, 和 `Boolean`（以大写字母开头）是合法的,但它们指的是一些特殊的内置类型,在你的代码中很少出现。对于类型, _始终_ 使用 `string`, `number`, 和 `boolean`。



## 数组

为了指定像 `[1, 2, 3]` 这样的数组类型，你可以使用 `number[]` 这种语法；这种语法适用于任何类型（例如，`string[]` 是一个字符串数组，以此类推）。
你也可能会看到这种写法 `Array<number>`，其含义是一样的。
当我们讲到 _泛型_ 时，将会更深入了解 `T<U>` 这种语法。

> 请注意，`[number]` 是另一种不同的东西；请参阅 [元组(Tuples)](/zh/docs/handbook/2/objects.html#tuple-types) 这一部分。

## `any`

TypeScript 也有一种特殊的类型: `any`，你可以在任何时候使用它，当你不希望某个特定的值引起类型检查错误时。

当一个值的类型为 `any` 时，你可以访问它的任何属性（它们也将是 `any` 类型）、像函数一样调用它、将它赋值给任何类型的值，或者其他任何语法上合法的操作：

```ts twoslash
let otherObj: any = { x: 0 };
// 下面的代码行都不会引发编译器错误。
// 使用 any 会禁用所有进一步的类型检查,
// 假定你比 TypeScript 更了解环境。
otherObj.foo();
otherObj();
otherObj.bar = 100;
otherObj = "hello";
const n: number = otherObj;
```

当你不想为了让 TypeScript 确认一行特定的代码是正确的而写出很长的类型声明时，`any` 类型就非常有用。

### `noImplicitAny`

当你没有指定类型，而 TypeScript 无法从上下文中推断出来时，编译器通常会默认为 `any` 类型。

然而，你通常会想避免这种情况，因为 `any` 不会被类型检查。使用编译器参数 `noImplicitAny`，将任何隐式的 `any` 标记为错误。

## 变量的类型注解

当你使用 `const`、`var` 或 `let` 声明一个变量时，你可以选择添加一个类型注解来明确指定变量的类型：

```ts twoslash
let myName: string = "Alice";
//        ^^^^^^^^ Type annotation
```

> TypeScript 不使用 “类型在左边”-样式的声明，如 `int x = 0`;
> 类型注解总是放在被类型化的对象的 _后面_。

然而，在大多数情况下，不需要这样做。
只要可能，TypeScript 会尝试自动 _推断_ 你的代码中的类型。例如，变量的类型根据其初始化器的类型进行推断：

```ts twoslash
// No type annotation needed -- 'myName' inferred as type 'string'
let myName = "Alice";
```

大部分情况下，你并不需要明确学习推断规则。
如果你刚开始接触，尝试使用比你想象中更少的类型注解 - 你可能会惊讶于TypeScript完全理解你的代码。

## 函数

函数是在JavaScript中传递数据的主要手段。
TypeScript允许你指定函数的输入和输出值的类型。

### 参数类型注解

当你声明一个函数时，可以在每个参数后添加类型注解，以声明函数接受的参数类型。
参数类型注解在参数名称之后：

```ts twoslash
// Parameter type annotation
function greet(name: string) {
  //                 ^^^^^^^^
  console.log("Hello, " + name.toUpperCase() + "!!");
}
```

当参数有类型注解时，传递给该函数的实参会进行类型检查：

```ts twoslash
// @errors: 2345
declare function greet(name: string): void;
// ---cut---
// Would be a runtime error if executed!
greet(42);
```

> 即使你没有在参数上添加类型注解，TypeScript 仍然会检查你是否传递了正确数量的参数。

### 返回值的类型注解

你也可以添加返回值的类型注解。
返回类型注解位于参数列表之后：

```ts twoslash
function getFavoriteNumber(): number {
  //                        ^^^^^^^^
  return 26;
}
```

与变量类型注释非常相似，您通常不需要返回类型注释，因为 TypeScript 会根据函数的 `return` 语句推断函数的返回类型。
上面示例中的类型注释不会改变任何内容。
某些代码库会出于文档目的显式指定返回类型，以防止意外更改或仅出于个人喜好。
#### 返回 Promise 的函数

如果你想注解一个返回 Promise 的函数的返回类型，你应该使用 `Promise` 类型：
```ts twoslash
async function getFavoriteNumber(): Promise<number> {
  return 26;
}
```

### 匿名函数

匿名函数与函数声明略有不同。
当函数出现在 TypeScript 可以确定如何调用它的位置时，该函数的参数将自动指定类型。

这是一个例子：

```ts twoslash
// @errors: 2551
const names = ["Alice", "Bob", "Eve"];

// 函数的上下文类型 -推断参数具有字符串类型
names.forEach(function (s) {
  console.log(s.toUpperCase());
});

// 上下文类型也适用于箭头函数
names.forEach((s) => {
  console.log(s.toUpperCase());
});
```

即使参数 `s` 没有类型注释，TypeScript 也通过 `forEach` 函数的类型以及数组推断出的类型来确定 `s` 的类型。
此过程称为 _上下文类型_，因为函数发生的 _上下文_ 决定了它应该具有什么类型。
与推理规则类似，您不需要显式地了解这是如何发生的，但了解它 _确实_ 会发生可以帮助您注意到何时不需要类型注释。
稍后，我们将看到更多示例，说明值出现的上下文如何影响其类型。
## 对象类型

除了原始类型之外，您遇到的最常见的类型是 _对象类型_。
这是指任何具有属性的 JavaScript 值，几乎是所有属性！
要定义对象类型，我们只需列出其属性及其类型。

例如，这是一个采用 point-like 的函数：

```ts twoslash
// 参数的类型注解是对象类型
function printCoord(pt: { x: number; y: number }) {
  //                      ^^^^^^^^^^^^^^^^^^^^^^^^
  console.log("The coordinate's x value is " + pt.x);
  console.log("The coordinate's y value is " + pt.y);
}
printCoord({ x: 3, y: 7 });
```

在这里，我们用一个类型来注解参数，该类型有两个属性 —— `x` 和 `y`，它们的类型都是 `number`。
你可以使用 `,` 或者 `;` 来分隔属性，最后一个分隔符是可选的。

每个属性的类型部分也是可选的。
如果你没有指定类型，那么将假定为 `any` 类型。

### 可选属性

对象类型也可以指定一些或者所有的属性为 _可选的_。
要实现这一点，只需在属性名称后添加 ?：

```ts twoslash
function printName(obj: { first: string; last?: string }) {
  // ...
}
// Both OK
printName({ first: "Bob" });
printName({ first: "Alice", last: "Alisson" });
```

在 JavaScript 中，如果你访问一个不存在的属性，你会得到 undefined 而不是运行时错误。
因此，当你从一个可选属性中 _读取_ 数据时，你必须在使用它之前检查是否为 undefined。
```ts twoslash
// @errors: 18048
function printName(obj: { first: string; last?: string }) {
  // 错误 -如果未提供“obj.last”，可能会崩溃！
  console.log(obj.last.toUpperCase());
  if (obj.last !== undefined) {
    // OK
    console.log(obj.last.toUpperCase());
  }

  // 使用现代 JavaScript 语法的安全替代方案：
  console.log(obj.last?.toUpperCase());
}
```

## 联合（Union）类型

TypeScript 的类型系统让你可以使用大量的运算符基于现有的类型构建新的类型。
既然我们已经知道如何写一些类型，那就是时候开始将他们以有趣的方式 _合并_ 了。

### 定义一个联合类型

你可能会看到的组合类型的第一种方式就是 _联合_ 类型。
联合类型是由两个或多个其他类型形成的类型，代表的值可能是这些类型中的 _任意一个_ 。
我们将这些类型中的每一个都称为该联合的 _成员_ 。

现在让我们写一个可以操作字符串或数字的函数：

```ts twoslash
// @errors: 2345
function printId(id: number | string) {
  console.log("Your ID is: " + id);
}
// OK
printId(101);
// OK
printId("202");
// Error
printId({ myID: 22342 });
```

### 使用联合类型

_提供_ 符合联合类型的值是很容易的 - 只需提供一个匹配联合类型任何成员的类型即可。
而如果你 _拥有_ 的是一个联合类型的值，那么你怎么处理它呢？

TypeScript 只有在操作对联合类型的 _每个_ 成员来说都是有效的，才会允许该操作。
例如，如果你有一个 `string | number` 的联合，那么你不能使用只在 `string` 上可用的方法：

```ts twoslash
// @errors: 2339
function printId(id: number | string) {
  console.log(id.toUpperCase());
}
```

解决方案是通过代码将联合类型进行 _收敛（narrow）_，就像在没有类型注解的 JavaScript 中那样。
所谓的 _收敛（narrow）_ 是指当 TypeScript 根据代码结构推断出一个值具有更具体类型的情况。

例如，TypeScript 知道只有 `string` 的 `typeof` 结果为 `"string"`：

```ts twoslash
function printId(id: number | string) {
  if (typeof id === "string") {
    // In this branch, id is of type 'string'
    console.log(id.toUpperCase());
  } else {
    // Here, id is of type 'number'
    console.log(id);
  }
}
```

另一个例子是使用像 `Array.isArray` 这样的函数：

```ts twoslash
function welcomePeople(x: string[] | string) {
  if (Array.isArray(x)) {
    // Here: 'x' is 'string[]'
    console.log("Hello, " + x.join(" and "));
  } else {
    // Here: 'x' is 'string'
    console.log("Welcome lone traveler " + x);
  }
}
```

注意，在 `else` 分支中，我们不需要做任何特殊的处理 - 如果 `x` 不是 `string[]`，那么它一定是 `string`。

有时候你会有一种联合，其中所有的成员都有一些共同之处。
例如，数组和字符串都有一个 `slice` 方法。
如果联合中的每个成员都有一个共同的属性，那么你可以在不进行收敛的情况下使用该属性：

```ts twoslash
// Return type is inferred as number[] | string
function getFirstThree(x: number[] | string) {
  return x.slice(0, 3);
}
```

> 可能会感到困惑，类型的 _联合_ 似乎具有这些类型属性的 _交集_。
> 这并非偶然 - _联合_ 这个名字源自类型理论。
> _联合_ `number | string` 是通过从每种类型中取 _值的联合_ 来合成的。
> 注意，给定两个具有相应事实的集合，只有这些事实的 _交集_ 适用于集合本身的 _联合_ 。 
> 例如，如果我们有一个房间里都是高个子戴着帽子的人，另一个房间里都是讲西班牙语并且戴着帽子的人，把这两个房间的人合在一起，我们所知道的 _每_ 个人唯一共同的事实就是他们都必须戴着帽子。

## 类型别名（Type Aliases）

在 TypeScript 中，我们经常直接在类型注释中使用对象类型和联合类型。
这种做法很方便，但通常我们会希望多次使用同一类型，并通过一个名称来引用它。

_类型别名_ 正是用于这种情况——为任何 _类型_ 提供一个 _名称_。
类型别名的语法如下：

```ts twoslash
type Point = {
  x: number;
  y: number;
};

// Exactly the same as the earlier example
function printCoord(pt: Point) {
  console.log("The coordinate's x value is " + pt.x);
  console.log("The coordinate's y value is " + pt.y);
}

printCoord({ x: 100, y: 100 });
```

实际上，您可以使用类型别名为任何类型命名，而不仅仅是对象类型。
例如，类型别名可以命名联合类型：

```ts twoslash
type ID = number | string;
```

请注意，别名 _只是_ 别名 -您不能使用类型别名来创建同一类型的不同 "版本"。
当您使用别名时，就像您编写了别名类型一样。
换句话说，这段代码可能 _看起来_ 非法，但根据 TypeScript 是可以的，因为两种类型都是同一类型的别名：

```ts twoslash
declare function getInput(): string;
declare function sanitize(str: string): string;
// ---cut---
type UserInputSanitizedString = string;

function sanitizeInput(str: string): UserInputSanitizedString {
  return sanitize(str);
}

// Create a sanitized input
let userInput = sanitizeInput(getInput());

// Can still be re-assigned with a string though
userInput = "new input";
```

## Interfaces

_Interface 声明_ 是命名对象类型的另一种方式：

```ts twoslash
interface Point {
  x: number;
  y: number;
}

function printCoord(pt: Point) {
  console.log("The coordinate's x value is " + pt.x);
  console.log("The coordinate's y value is " + pt.y);
}

printCoord({ x: 100, y: 100 });
```

就像我们上面使用类型别名一样，该示例的工作方式就像我们使用匿名对象类型一样。
TypeScript 只关心我们传递给 `printCoord` 的值的 _结构_ -它只关心它是否具有预期的属性。
只关心类型的结构和功能就是我们将 TypeScript 称为 _结构类型_ 类型系统的原因。

### Type Aliases 和 Interface 之间的差异

类型别名和接口非常相似，在许多情况下，你可以自由地在它们之间进行选择。
`interface` 的所有特性几乎在 `type` 中都有，关键的区别在于， `type` 一旦定义，不能重新声明以添加新的属性，而 `interface` 则总是可以扩展的。

<div class='table-container'>
<table class='full-width-table'>
  <tbody>
    <tr>
      <th><code>Interface</code></th>
      <th><code>Type</code></th>
    </tr>
    <tr>
      <td>
        <p>Extending an interface</p>
        <code><pre>
interface Animal {
  name: string;
}<br/>
interface Bear extends Animal {
  honey: boolean;
}<br/>
const bear = getBear();
bear.name;
bear.honey;
        </pre></code>
      </td>
      <td>
        <p>Extending a type via intersections</p>
        <code><pre>
type Animal = {
  name: string;
}<br/>
type Bear = Animal & { 
  honey: boolean;
}<br/>
const bear = getBear();
bear.name;
bear.honey;
        </pre></code>
      </td>
    </tr>
    <tr>
      <td>
        <p>Adding new fields to an existing interface</p>
        <code><pre>
interface Window {
  title: string;
}<br/>
interface Window {
  ts: TypeScriptAPI;
}<br/>
const src = 'const a = "Hello World"';
window.ts.transpileModule(src, {});
        </pre></code>
      </td>
      <td>
        <p>A type cannot be changed after being created</p>
        <code><pre>
type Window = {
  title: string;
}<br/>
type Window = {
  ts: TypeScriptAPI;
}<br/>
<span style="color: #A31515"> // Error: Duplicate identifier 'Window'.</span><br/>
        </pre></code>
      </td>
    </tr>
    </tbody>
</table>
</div>

您将在后面的章节中了解有关这些概念的更多信息，因此如果您不能立即理解所有这些概念，请不要担心。

- Prior to TypeScript version 4.2, type alias names [_may_ appear in error messages](/zh/play?#code/PTAEGEHsFsAcEsA2BTATqNrLusgzngIYDm+oA7koqIYuYQJ56gCueyoAUCKAC4AWHAHaFcoSADMaQ0PCG80EwgGNkALk6c5C1EtWgAsqOi1QAb06groEbjWg8vVHOKcAvpokshy3vEgyyMr8kEbQJogAFND2YREAlOaW1soBeJAoAHSIkMTRmbbI8e6aPMiZxJmgACqCGKhY6ABGyDnkFFQ0dIzMbBwCwqIccabcYLyQoKjIEmh8kwN8DLAc5PzwwbLMyAAeK77IACYaQSEjUWZWhfYAjABMAMwALA+gbsVjoADqgjKESytQPxCHghAByXigYgBfr8LAsYj8aQMUASbDQcRSExCeCwFiIQh+AKfAYyBiQFgOPyIaikSGLQo0Zj-aazaY+dSaXjLDgAGXgAC9CKhDqAALxJaw2Ib2RzOISuDycLw+ImBYKQflCkWRRD2LXCw6JCxS1JCdJZHJ5RAFIbFJU8ADKC3WzEcnVZaGYE1ABpFnFOmsFhsil2uoHuzwArO9SmAAEIsSFrZB-GgAjjA5gtVN8VCEc1o1C4Q4AGlR2AwO1EsBQoAAbvB-gJ4HhPgB5aDwem-Ph1TCV3AEEirTp4ELtRbTPD4vwKjOfAuioSQHuDXBcnmgACC+eCONFEs73YAPGGZVT5cRyyhiHh7AAON7lsG3vBggB8XGV3l8-nVISOgghxoLq9i7io-AHsayRWGaFrlFauq2rg9qaIGQHwCBqChtKdgRo8TxRjeyB3o+7xAA), sometimes in place of the equivalent anonymous type (which may or may not be desirable). Interfaces will always be named in error messages.
- Type aliases may not participate [in declaration merging, but interfaces can](/zh/play?#code/PTAEEEDtQS0gXApgJwGYEMDGjSfdAIx2UQFoB7AB0UkQBMAoEUfO0Wgd1ADd0AbAK6IAzizp16ALgYM4SNFhwBZdAFtV-UAG8GoPaADmNAcMmhh8ZHAMMAvjLkoM2UCvWad+0ARL0A-GYWVpA29gyY5JAWLJAwGnxmbvGgALzauvpGkCZmAEQAjABMAMwALLkANBl6zABi6DB8okR4Jjg+iPSgABboovDk3jjo5pbW1d6+dGb5djLwAJ7UoABKiJTwjThpnpnGpqPBoTLMAJrkArj4kOTwYmycPOhW6AR8IrDQ8N04wmo4HHQCwYi2Waw2W1S6S8HX8gTGITsQA).
- Interfaces may only be used to [declare the shapes of objects, not rename primitives](/zh/play?#code/PTAEAkFMCdIcgM6gC4HcD2pIA8CGBbABwBtIl0AzUAKBFAFcEBLAOwHMUBPQs0XFgCahWyGBVwBjMrTDJMAshOhMARpD4tQ6FQCtIE5DWoixk9QEEWAeV37kARlABvaqDegAbrmL1IALlAEZGV2agBfampkbgtrWwMAJlAAXmdXdy8ff0Dg1jZwyLoAVWZ2Lh5QVHUJflAlSFxROsY5fFAWAmk6CnRoLGwmILzQQmV8JmQmDzI-SOiKgGV+CaYAL0gBBdyy1KCQ-Pn1AFFplgA5enw1PtSWS+vCsAAVAAtB4QQWOEMKBuYVUiVCYvYQsUTQcRSBDGMGmKSgAAa-VEgiQe2GLgKQA).
- Interface names will [_always_ appear in their original form](/play?#code/PTAEGEHsFsAcEsA2BTATqNrLusgzngIYDm+oA7koqIYuYQJ56gCueyoAUCKAC4AWHAHaFcoSADMaQ0PCG80EwgGNkALk6c5C1EtWgAsqOi1QAb06groEbjWg8vVHOKcAvpokshy3vEgyyMr8kEbQJogAFND2YREAlOaW1soBeJAoAHSIkMTRmbbI8e6aPMiZxJmgACqCGKhY6ABGyDnkFFQ0dIzMbBwCwqIccabcYLyQoKjIEmh8kwN8DLAc5PzwwbLMyAAeK77IACYaQSEjUWY2Q-YAjABMAMwALA+gbsVjNXW8yxySoAADaAA0CCaZbPh1XYqXgOIY0ZgmcK0AA0nyaLFhhGY8F4AHJmEJILCWsgZId4NNfIgGFdcIcUTVfgBlZTOWC8T7kAJ42G4eT+GS42QyRaYbCgXAEEguTzeXyCjDBSAAQSE8Ai0Xsl0K9kcziExDeiQs1lAqSE6SyOTy0AKQ2KHk4p1V6s1OuuoHuzwArMagA) in error messages, but _only_ when they are used by name.
- Using interfaces with `extends` [can often be more performant for the compiler](https://github.com/microsoft/TypeScript/wiki/Performance#preferring-interfaces-over-intersections) than type aliases with intersections

大体上，你可以根据个人喜好进行选择，如果TypeScript需要其他种类的声明，它会告诉你。如果你想要一个启发式的规则，那就是先用 `interface`，直到你需要用到 `type` 的特性。

## 类型断言

有时您会获得 TypeScript 无法了解的值类型信息。

例如，如果您使用的是 `document.getElementById`，TypeScript 只知道这将返回某种 `HTMLElement`，但您可能知道您的页面始终会有一个具有给定 ID 的 `HTMLCanvasElement`。

在这种情况下，您可以使用 _type断言_ 来指定更具体的类型：

```ts twoslash
const myCanvas = document.getElementById("main_canvas") as HTMLCanvasElement;
```

与类型注释一样，类型断言会被编译器删除，并且不会影响代码的运行时行为。

您还可以使用尖括号语法（除非代码位于 `.tsx` 文件中），它等效于：

```ts twoslash
const myCanvas = <HTMLCanvasElement>document.getElementById("main_canvas");
```

> 提醒：由于类型断言在编译时被删除，因此不存在与类型断言相关的运行时检查。
> 如果类型断言错误，则不会生成异常或 `null`。

TypeScript 只允许类型断言转换为 _更具体_ 或 _不太具体_ 类型的版本。
该规则可防止 "不可能" 的强制，例如：

```ts twoslash
// @errors: 2352
const x = "hello" as number;
```

有时，此规则可能过于保守，并且不允许可能有效的更复杂的强制。
如果发生这种情况，您可以使用两个断言，首先是 `any`（或 `unknown`，我们稍后将介绍），然后是所需的类型：

```ts twoslash
declare const expr: any;
type T = { a: 1; b: 2; c: 3 };
// ---cut---
const a = expr as any as T;
```

## 字面量类型

除了一般类型 `string` 和 `number` 之外，我们还可以在类型位置引用 _特定_ 字符串和数字。

理解这个问题的一个方式是考虑 JavaScript 提供了不同的声明变量的方式。 `var` 和 `let` 允许改变变量内的值，而 const 则不允许。这在 TypeScript 创建字面量类型时得到了体现。

```ts twoslash
let changingString = "Hello World";
changingString = "Olá Mundo";
// Because `changingString` can represent any possible string, that
// is how TypeScript describes it in the type system
changingString;
// ^?

const constantString = "Hello World";
// Because `constantString` can only represent 1 possible string, it
// has a literal type representation
constantString;
// ^?
```

就其本身而言，字面量类型并不是很有价值：

```ts twoslash
// @errors: 2322
let x: "hello" = "hello";
// OK
x = "hello";
// ...
x = "howdy";
```

一个只能有一个值的变量并没有多大用处！

但是通过将文字 _组合_ 成联合类型，您可以表达一个更有用的概念 -例如，只接受一组特定已知值的函数：

```ts twoslash
// @errors: 2345
function printText(s: string, alignment: "left" | "right" | "center") {
  // ...
}
printText("Hello, world", "left");
printText("G'day, mate", "centre");
```

数字字面量类型的工作方式相同：

```ts twoslash
function compare(a: string, b: string): -1 | 0 | 1 {
  return a === b ? 0 : a > b ? 1 : -1;
}
```

当然，您可以将它们与非自变量类型结合起来：

```ts twoslash
// @errors: 2345
interface Options {
  width: number;
}
function configure(x: Options | "auto") {
  // ...
}
configure({ width: 100 });
configure("auto");
configure("automatic");
```

还有另一种字面量类型：布尔字面量。
在 TypeScript 中，只有两种布尔字面量类型，正如你所猜测的那样，它们是 `true` 和 `false` 类型。
实际上，`boolean` 类型本身就是 `true | false` 联合类型的别名。

### 字面量类型推断

当你用一个对象初始化一个变量时，TypeScript假设对象的那些属性可能会在后面改变值。
例如，如果你编写了如下代码：

```ts twoslash
declare const someCondition: boolean;
// ---cut---
const obj = { counter: 0 };
if (someCondition) {
  obj.counter = 1;
}
```

TypeScript 不会假设把 `1` 分配给一个原本值为 `0` 的字段是错误的。
换句话说，`obj.counter` 的类型必须是 `number`，而不是 `0`，因为类型用来确定 _读取（reading）_ 和 _写入（writing）_ 的行为。

这对于字符串也同样适用：

```ts twoslash
// @errors: 2345
declare function handleRequest(url: string, method: "GET" | "POST"): void;

const req = { url: "https://example.com", method: "GET" };
handleRequest(req.url, req.method);
```

在上述示例中，`req.method` 被推断为 `string` 类型，而不是 `"GET"`。因为在创建 `req` 对象和调用 `handleRequest` 函数之间，代码可能会执行，这期间可能会有新的字符串如 `"GUESS"` 赋值给 `req.method`。因此，TypeScript 认为这段代码存在错误。

有两种方法可以解决这个问题：

1. 你可以通过在任何一个位置添加类型断言来改变类型推断：

   ```ts twoslash
   declare function handleRequest(url: string, method: "GET" | "POST"): void;
   // ---cut---
   // 方法一：
   const req = { url: "https://example.com", method: "GET" as "GET" };
   // 方法二：
   handleRequest(req.url, req.method as "GET");
   ```

   方法一的意思是 “我打算让 `req.method` 始终具有 _字面量类型_ `"GET"`”，这样可以防止后续将 `"GUESS"` 赋值给该字段。
   方法二的意思是 “我出于其他原因知道 `req.method` 的值是 `"GET"`”。

2. 你可以使用 `as const` 将整个对象转换为字面量类型：

   ```ts twoslash
   declare function handleRequest(url: string, method: "GET" | "POST"): void;
   // ---cut---
   const req = { url: "https://example.com", method: "GET" } as const;
   handleRequest(req.url, req.method);
   ```

`as const` 后缀的作用类似于 `const`，但它是针对类型系统的。它确保所有属性都被赋予字面量类型，而不是像 `string` 或 `number` 这样的更通用版本。

## `null` 和 `undefined`

JavaScript 有两个原始值用来表示缺失或未初始化的值：`null` 和 `undefined`。

TypeScript 有两个相应的 _类型_，名字也是 `null` 和 `undefined`。这些类型的行为取决于是否开启了 [`strictNullChecks`](/zh/tsconfig#strictNullChecks) 选项。

### 关闭 `strictNullChecks` 

当 [`strictNullChecks`](/zh/tsconfig#strictNullChecks) _关闭_ 时，可能为 `null` 或 `undefined` 的值仍然可以正常访问，而且 `null` 和 `undefined` 可以被赋值给任何类型的属性。
这与没有空值检查的语言（例如 C#、Java）的行为类似。
不检查这些值往往是导致大量错误的主要原因；我们总是建议人们在实际可行的情况下开启 [`strictNullChecks`](/zh/tsconfig#strictNullChecks) 

### 开启 `strictNullChecks`

当 _开启_ [`strictNullChecks`](/zh/tsconfig#strictNullChecks) 时，如果一个值是 `null` 或 `undefined`，在使用该值的方法或属性之前，你需要先检查这些值是否为 `null` 或 `undefined`。
就像在使用可选属性之前检查 `undefined` 一样，我们可以使用 _收敛（narrowing）_ 类型范围来检查可能为 null 的值：

```ts twoslash
function doSomething(x: string | null) {
  if (x === null) {
    // do nothing
  } else { // x 已经被收敛为 string 类型，可以安全使用
    console.log("Hello, " + x.toUpperCase());
  }
}
```

### 非空断言操作符（后缀 `!`）

在 TypeScript 中，也有一种特殊的语法用于在不进行任何显式检查的情况下从类型中移除 `null` 和 `undefined`。
在任何表达式后写 `!` 实际上是在断言该值不是 `null` 或 `undefined`：

```ts twoslash
function liveDangerously(x?: number | null) {
  // No error
  console.log(x!.toFixed());
}
```

就像其他类型断言一样，这并不会改变你代码的运行时行为，所以只有当你知道值 _确定不会_ 是 `null` 或 `undefined` 时，才应使用 `!`。

## 枚举（Enums）

枚举（Enums）是 TypeScript 为 JavaScript 添加的一项功能，它允许描述一个值可以是一组可能的命名常量中的一个。与 TypeScript 的大部分特性不同，这 _不是_ JavaScript 的类型级别的增加，而是添加到了语言和运行时中。因为这个原因，这是一个你应该知道存在的特性，但在你特别了解之前先尽量少使用枚举。你可以在[枚举参考页面](/zh/docs/handbook/enums.html)中阅读更多关于枚举的内容。

## 较少使用的原始类型

有必要提及在类型系统中表示的 JavaScript 的其余原始类型，
尽管我们在这里不会深入探讨。

#### `bigint`

从 ES2020 开始，JavaScript 中有一个用于表示非常大的整数的原始类型，`BigInt`:

```ts twoslash
// @target: es2020

// Creating a bigint via the BigInt function
const oneHundred: bigint = BigInt(100);

// Creating a BigInt via the literal syntax
const anotherHundred: bigint = 100n;
```

你可以在 [TypeScript 3.2 发布笔记](/zh/docs/handbook/release-notes/typescript-3-2.html#bigint) 中了解更多关于 BigInt 的信息。

#### `symbol`

JavaScript 中有一个原始类型，用于通过函数“Symbol()”创建全局唯一引用：
```ts twoslash
// @errors: 2367
const firstName = Symbol("name");
const secondName = Symbol("name");

if (firstName === secondName) {
  // Can't ever happen
}
```

您可以在[`symbol` 参考页面](/zh/docs/handbook/symbols.html) 中了解有关它们的更多信息。
