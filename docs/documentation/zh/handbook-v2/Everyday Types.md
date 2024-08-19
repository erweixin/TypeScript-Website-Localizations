---
title: 日常类型
layout: docs
permalink: /zh/docs/handbook/2/everyday-types.html
oneline: "typeScript 的原始类型"
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

## Union Types

TypeScript's type system allows you to build new types out of existing ones using a large variety of operators.
Now that we know how to write a few types, it's time to start _combining_ them in interesting ways.

### Defining a Union Type

The first way to combine types you might see is a _union_ type.
A union type is a type formed from two or more other types, representing values that may be _any one_ of those types.
We refer to each of these types as the union's _members_.

Let's write a function that can operate on strings or numbers:

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

### Working with Union Types

It's easy to _provide_ a value matching a union type - simply provide a type matching any of the union's members.
If you _have_ a value of a union type, how do you work with it?

TypeScript will only allow an operation if it is valid for _every_ member of the union.
For example, if you have the union `string | number`, you can't use methods that are only available on `string`:

```ts twoslash
// @errors: 2339
function printId(id: number | string) {
  console.log(id.toUpperCase());
}
```

The solution is to _narrow_ the union with code, the same as you would in JavaScript without type annotations.
_Narrowing_ occurs when TypeScript can deduce a more specific type for a value based on the structure of the code.

For example, TypeScript knows that only a `string` value will have a `typeof` value `"string"`:

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

Another example is to use a function like `Array.isArray`:

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

Notice that in the `else` branch, we don't need to do anything special - if `x` wasn't a `string[]`, then it must have been a `string`.

Sometimes you'll have a union where all the members have something in common.
For example, both arrays and strings have a `slice` method.
If every member in a union has a property in common, you can use that property without narrowing:

```ts twoslash
// Return type is inferred as number[] | string
function getFirstThree(x: number[] | string) {
  return x.slice(0, 3);
}
```

> It might be confusing that a _union_ of types appears to have the _intersection_ of those types' properties.
> This is not an accident - the name _union_ comes from type theory.
> The _union_ `number | string` is composed by taking the union _of the values_ from each type.
> Notice that given two sets with corresponding facts about each set, only the _intersection_ of those facts applies to the _union_ of the sets themselves.
> For example, if we had a room of tall people wearing hats, and another room of Spanish speakers wearing hats, after combining those rooms, the only thing we know about _every_ person is that they must be wearing a hat.

## Type Aliases

We've been using object types and union types by writing them directly in type annotations.
This is convenient, but it's common to want to use the same type more than once and refer to it by a single name.

A _type alias_ is exactly that - a _name_ for any _type_.
The syntax for a type alias is:

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

You can actually use a type alias to give a name to any type at all, not just an object type.
For example, a type alias can name a union type:

```ts twoslash
type ID = number | string;
```

Note that aliases are _only_ aliases - you cannot use type aliases to create different/distinct "versions" of the same type.
When you use the alias, it's exactly as if you had written the aliased type.
In other words, this code might _look_ illegal, but is OK according to TypeScript because both types are aliases for the same type:

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

An _interface declaration_ is another way to name an object type:

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

Just like when we used a type alias above, the example works just as if we had used an anonymous object type.
TypeScript is only concerned with the _structure_ of the value we passed to `printCoord` - it only cares that it has the expected properties.
Being concerned only with the structure and capabilities of types is why we call TypeScript a _structurally typed_ type system.

### Differences Between Type Aliases and Interfaces

Type aliases and interfaces are very similar, and in many cases you can choose between them freely.
Almost all features of an `interface` are available in `type`, the key distinction is that a type cannot be re-opened to add new properties vs an interface which is always extendable.

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

You'll learn more about these concepts in later chapters, so don't worry if you don't understand all of these right away.

- Prior to TypeScript version 4.2, type alias names [_may_ appear in error messages](/play?#code/PTAEGEHsFsAcEsA2BTATqNrLusgzngIYDm+oA7koqIYuYQJ56gCueyoAUCKAC4AWHAHaFcoSADMaQ0PCG80EwgGNkALk6c5C1EtWgAsqOi1QAb06groEbjWg8vVHOKcAvpokshy3vEgyyMr8kEbQJogAFND2YREAlOaW1soBeJAoAHSIkMTRmbbI8e6aPMiZxJmgACqCGKhY6ABGyDnkFFQ0dIzMbBwCwqIccabcYLyQoKjIEmh8kwN8DLAc5PzwwbLMyAAeK77IACYaQSEjUWZWhfYAjABMAMwALA+gbsVjoADqgjKESytQPxCHghAByXigYgBfr8LAsYj8aQMUASbDQcRSExCeCwFiIQh+AKfAYyBiQFgOPyIaikSGLQo0Zj-aazaY+dSaXjLDgAGXgAC9CKhDqAALxJaw2Ib2RzOISuDycLw+ImBYKQflCkWRRD2LXCw6JCxS1JCdJZHJ5RAFIbFJU8ADKC3WzEcnVZaGYE1ABpFnFOmsFhsil2uoHuzwArO9SmAAEIsSFrZB-GgAjjA5gtVN8VCEc1o1C4Q4AGlR2AwO1EsBQoAAbvB-gJ4HhPgB5aDwem-Ph1TCV3AEEirTp4ELtRbTPD4vwKjOfAuioSQHuDXBcnmgACC+eCONFEs73YAPGGZVT5cRyyhiHh7AAON7lsG3vBggB8XGV3l8-nVISOgghxoLq9i7io-AHsayRWGaFrlFauq2rg9qaIGQHwCBqChtKdgRo8TxRjeyB3o+7xAA), sometimes in place of the equivalent anonymous type (which may or may not be desirable). Interfaces will always be named in error messages.
- Type aliases may not participate [in declaration merging, but interfaces can](/play?#code/PTAEEEDtQS0gXApgJwGYEMDGjSfdAIx2UQFoB7AB0UkQBMAoEUfO0Wgd1ADd0AbAK6IAzizp16ALgYM4SNFhwBZdAFtV-UAG8GoPaADmNAcMmhh8ZHAMMAvjLkoM2UCvWad+0ARL0A-GYWVpA29gyY5JAWLJAwGnxmbvGgALzauvpGkCZmAEQAjABMAMwALLkANBl6zABi6DB8okR4Jjg+iPSgABboovDk3jjo5pbW1d6+dGb5djLwAJ7UoABKiJTwjThpnpnGpqPBoTLMAJrkArj4kOTwYmycPOhW6AR8IrDQ8N04wmo4HHQCwYi2Waw2W1S6S8HX8gTGITsQA).
- Interfaces may only be used to [declare the shapes of objects, not rename primitives](/play?#code/PTAEAkFMCdIcgM6gC4HcD2pIA8CGBbABwBtIl0AzUAKBFAFcEBLAOwHMUBPQs0XFgCahWyGBVwBjMrTDJMAshOhMARpD4tQ6FQCtIE5DWoixk9QEEWAeV37kARlABvaqDegAbrmL1IALlAEZGV2agBfampkbgtrWwMAJlAAXmdXdy8ff0Dg1jZwyLoAVWZ2Lh5QVHUJflAlSFxROsY5fFAWAmk6CnRoLGwmILzQQmV8JmQmDzI-SOiKgGV+CaYAL0gBBdyy1KCQ-Pn1AFFplgA5enw1PtSWS+vCsAAVAAtB4QQWOEMKBuYVUiVCYvYQsUTQcRSBDGMGmKSgAAa-VEgiQe2GLgKQA).
- Interface names will [_always_ appear in their original form](/play?#code/PTAEGEHsFsAcEsA2BTATqNrLusgzngIYDm+oA7koqIYuYQJ56gCueyoAUCKAC4AWHAHaFcoSADMaQ0PCG80EwgGNkALk6c5C1EtWgAsqOi1QAb06groEbjWg8vVHOKcAvpokshy3vEgyyMr8kEbQJogAFND2YREAlOaW1soBeJAoAHSIkMTRmbbI8e6aPMiZxJmgACqCGKhY6ABGyDnkFFQ0dIzMbBwCwqIccabcYLyQoKjIEmh8kwN8DLAc5PzwwbLMyAAeK77IACYaQSEjUWY2Q-YAjABMAMwALA+gbsVjNXW8yxySoAADaAA0CCaZbPh1XYqXgOIY0ZgmcK0AA0nyaLFhhGY8F4AHJmEJILCWsgZId4NNfIgGFdcIcUTVfgBlZTOWC8T7kAJ42G4eT+GS42QyRaYbCgXAEEguTzeXyCjDBSAAQSE8Ai0Xsl0K9kcziExDeiQs1lAqSE6SyOTy0AKQ2KHk4p1V6s1OuuoHuzwArMagA) in error messages, but _only_ when they are used by name.
- Using interfaces with `extends` [can often be more performant for the compiler](https://github.com/microsoft/TypeScript/wiki/Performance#preferring-interfaces-over-intersections) than type aliases with intersections

For the most part, you can choose based on personal preference, and TypeScript will tell you if it needs something to be the other kind of declaration. If you would like a heuristic, use `interface` until you need to use features from `type`.

## Type Assertions

Sometimes you will have information about the type of a value that TypeScript can't know about.

For example, if you're using `document.getElementById`, TypeScript only knows that this will return _some_ kind of `HTMLElement`, but you might know that your page will always have an `HTMLCanvasElement` with a given ID.

In this situation, you can use a _type assertion_ to specify a more specific type:

```ts twoslash
const myCanvas = document.getElementById("main_canvas") as HTMLCanvasElement;
```

Like a type annotation, type assertions are removed by the compiler and won't affect the runtime behavior of your code.

You can also use the angle-bracket syntax (except if the code is in a `.tsx` file), which is equivalent:

```ts twoslash
const myCanvas = <HTMLCanvasElement>document.getElementById("main_canvas");
```

> Reminder: Because type assertions are removed at compile-time, there is no runtime checking associated with a type assertion.
> There won't be an exception or `null` generated if the type assertion is wrong.

TypeScript only allows type assertions which convert to a _more specific_ or _less specific_ version of a type.
This rule prevents "impossible" coercions like:

```ts twoslash
// @errors: 2352
const x = "hello" as number;
```

Sometimes this rule can be too conservative and will disallow more complex coercions that might be valid.
If this happens, you can use two assertions, first to `any` (or `unknown`, which we'll introduce later), then to the desired type:

```ts twoslash
declare const expr: any;
type T = { a: 1; b: 2; c: 3 };
// ---cut---
const a = expr as any as T;
```

## Literal Types

In addition to the general types `string` and `number`, we can refer to _specific_ strings and numbers in type positions.

One way to think about this is to consider how JavaScript comes with different ways to declare a variable. Both `var` and `let` allow for changing what is held inside the variable, and `const` does not. This is reflected in how TypeScript creates types for literals.

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

By themselves, literal types aren't very valuable:

```ts twoslash
// @errors: 2322
let x: "hello" = "hello";
// OK
x = "hello";
// ...
x = "howdy";
```

It's not much use to have a variable that can only have one value!

But by _combining_ literals into unions, you can express a much more useful concept - for example, functions that only accept a certain set of known values:

```ts twoslash
// @errors: 2345
function printText(s: string, alignment: "left" | "right" | "center") {
  // ...
}
printText("Hello, world", "left");
printText("G'day, mate", "centre");
```

Numeric literal types work the same way:

```ts twoslash
function compare(a: string, b: string): -1 | 0 | 1 {
  return a === b ? 0 : a > b ? 1 : -1;
}
```

Of course, you can combine these with non-literal types:

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

There's one more kind of literal type: boolean literals.
There are only two boolean literal types, and as you might guess, they are the types `true` and `false`.
The type `boolean` itself is actually just an alias for the union `true | false`.

### Literal Inference

When you initialize a variable with an object, TypeScript assumes that the properties of that object might change values later.
For example, if you wrote code like this:

```ts twoslash
declare const someCondition: boolean;
// ---cut---
const obj = { counter: 0 };
if (someCondition) {
  obj.counter = 1;
}
```

TypeScript doesn't assume the assignment of `1` to a field which previously had `0` is an error.
Another way of saying this is that `obj.counter` must have the type `number`, not `0`, because types are used to determine both _reading_ and _writing_ behavior.

The same applies to strings:

```ts twoslash
// @errors: 2345
declare function handleRequest(url: string, method: "GET" | "POST"): void;

const req = { url: "https://example.com", method: "GET" };
handleRequest(req.url, req.method);
```

In the above example `req.method` is inferred to be `string`, not `"GET"`. Because code can be evaluated between the creation of `req` and the call of `handleRequest` which could assign a new string like `"GUESS"` to `req.method`, TypeScript considers this code to have an error.

There are two ways to work around this.

1. You can change the inference by adding a type assertion in either location:

   ```ts twoslash
   declare function handleRequest(url: string, method: "GET" | "POST"): void;
   // ---cut---
   // Change 1:
   const req = { url: "https://example.com", method: "GET" as "GET" };
   // Change 2
   handleRequest(req.url, req.method as "GET");
   ```

   Change 1 means "I intend for `req.method` to always have the _literal type_ `"GET"`", preventing the possible assignment of `"GUESS"` to that field after.
   Change 2 means "I know for other reasons that `req.method` has the value `"GET"`".

2. You can use `as const` to convert the entire object to be type literals:

   ```ts twoslash
   declare function handleRequest(url: string, method: "GET" | "POST"): void;
   // ---cut---
   const req = { url: "https://example.com", method: "GET" } as const;
   handleRequest(req.url, req.method);
   ```

The `as const` suffix acts like `const` but for the type system, ensuring that all properties are assigned the literal type instead of a more general version like `string` or `number`.

## `null` and `undefined`

JavaScript has two primitive values used to signal absent or uninitialized value: `null` and `undefined`.

TypeScript has two corresponding _types_ by the same names. How these types behave depends on whether you have the [`strictNullChecks`](/tsconfig#strictNullChecks) option on.

### `strictNullChecks` off

With [`strictNullChecks`](/tsconfig#strictNullChecks) _off_, values that might be `null` or `undefined` can still be accessed normally, and the values `null` and `undefined` can be assigned to a property of any type.
This is similar to how languages without null checks (e.g. C#, Java) behave.
The lack of checking for these values tends to be a major source of bugs; we always recommend people turn [`strictNullChecks`](/tsconfig#strictNullChecks) on if it's practical to do so in their codebase.

### `strictNullChecks` on

With [`strictNullChecks`](/tsconfig#strictNullChecks) _on_, when a value is `null` or `undefined`, you will need to test for those values before using methods or properties on that value.
Just like checking for `undefined` before using an optional property, we can use _narrowing_ to check for values that might be `null`:

```ts twoslash
function doSomething(x: string | null) {
  if (x === null) {
    // do nothing
  } else {
    console.log("Hello, " + x.toUpperCase());
  }
}
```

### Non-null Assertion Operator (Postfix `!`)

TypeScript also has a special syntax for removing `null` and `undefined` from a type without doing any explicit checking.
Writing `!` after any expression is effectively a type assertion that the value isn't `null` or `undefined`:

```ts twoslash
function liveDangerously(x?: number | null) {
  // No error
  console.log(x!.toFixed());
}
```

Just like other type assertions, this doesn't change the runtime behavior of your code, so it's important to only use `!` when you know that the value _can't_ be `null` or `undefined`.

## Enums

Enums are a feature added to JavaScript by TypeScript which allows for describing a value which could be one of a set of possible named constants. Unlike most TypeScript features, this is _not_ a type-level addition to JavaScript but something added to the language and runtime. Because of this, it's a feature which you should know exists, but maybe hold off on using unless you are sure. You can read more about enums in the [Enum reference page](/docs/handbook/enums.html).

## Less Common Primitives

It's worth mentioning the rest of the primitives in JavaScript which are represented in the type system.
Though we will not go into depth here.

#### `bigint`

From ES2020 onwards, there is a primitive in JavaScript used for very large integers, `BigInt`:

```ts twoslash
// @target: es2020

// Creating a bigint via the BigInt function
const oneHundred: bigint = BigInt(100);

// Creating a BigInt via the literal syntax
const anotherHundred: bigint = 100n;
```

You can learn more about BigInt in [the TypeScript 3.2 release notes](/docs/handbook/release-notes/typescript-3-2.html#bigint).

#### `symbol`

There is a primitive in JavaScript used to create a globally unique reference via the function `Symbol()`:

```ts twoslash
// @errors: 2367
const firstName = Symbol("name");
const secondName = Symbol("name");

if (firstName === secondName) {
  // Can't ever happen
}
```

You can learn more about them in [Symbols reference page](/docs/handbook/symbols.html).
