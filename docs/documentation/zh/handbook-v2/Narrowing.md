---
title: 类型收敛
layout: docs
permalink: /zh/docs/handbook/2/narrowing.html
oneline: "了解 TypeScript 如何使用 JavaScript 知识来减少项目中的类型语法数量。"
---

假设我们有一个名为 `padLeft` 的函数。

```ts twoslash
function padLeft(padding: number | string, input: string): string {
  throw new Error("Not implemented yet!");
}
```

如果 `padding` 是一个 `number`，那么会将其视为我们希望在 `input` 前面添加的空格数量。
如果 `padding` 是一个 `string`，那么应该将 `padding` 直接添加到 `input` 的前面。
现在，让我们尝试实现当 `padLeft` 传递给 `padding` 的参数是 `number` 类型时的逻辑。

```ts twoslash
// @errors: 2345
function padLeft(padding: number | string, input: string): string {
  return " ".repeat(padding) + input;
}
```

呀，我们在 `padding` 上遇到了一个错误。
TypeScript正在警告我们，我们向 `repeat` 函数传递了一个类型为 `number | string` 的值，但这个函数只接受 `number` 类型，这是正确的。
换言之，我们还没有明确地检查 `padding` 是否是一个 `number`，也没有处理它是 `string` 的情况，所以我们应该这样做。

```ts twoslash
function padLeft(padding: number | string, input: string): string {
  if (typeof padding === "number") {
    return " ".repeat(padding) + input;
  }
  return padding + input;
}
```

如果这看起来大部分像不太有趣的 JavaScript 代码，那也是我们的目的之一。
除了我们设置的注释，这份 TypeScript 代码看起来就像 JavaScript。
其思想是，TypeScript 的类型系统旨在使编写典型的 JavaScript 代码尽可能简单，无需过于费力就能实现类型安全。

虽然这可能看起来并无特别，但在幕后，实际上有很多事情在进行。
就像 TypeScript 使用静态类型分析运行时值一样，它在 JavaScript 的运行时控制流程构造（如 `if/else`，条件三元运算符，循环，真值检查等）中进行类型分析，这些都会影响那些类型。

在我们的 `if` 检查中，TypeScript 会看到 `typeof padding === \"number\"`，并理解这是一种叫做 _类型守卫（type guard）_ 的特殊代码。
TypeScript 跟踪我们的程序可能执行的路径，分析在给定位置的值的最具体的可能类型。
它查看这些特殊检查（被称为 _类型守卫_ ）和赋值，对比已声明的更具体的类型，这个过程被称为 _收敛_。
在许多编辑器中，我们可以观察到这些类型是如何变化的，我们甚至还会在我们的例子中进行此类操作。

```ts twoslash
function padLeft(padding: number | string, input: string): string {
  if (typeof padding === "number") {
    return " ".repeat(padding) + input;
    //                ^?
  }
  return padding + input;
  //     ^?
}
```

TypeScript 用于类型收敛的语句有几种不同的构造方式。

## `typeof` 类型守卫

正如我们所见，JavaScript 支持一个 `typeof` 操作符，它可以在运行时给我们提供关于值类型的基础信息。
TypeScript 期望这个操作符返回一组特定的字符串：

- `"string"`
- `"number"`
- `"bigint"`
- `"boolean"`
- `"symbol"`
- `"undefined"`
- `"object"`
- `"function"`

"正如我们在 `padLeft` 中看到的，这个操作符在许多 JavaScript 库中经常出现，而且 TypeScript 可以理解它在不同分支中如何进行类型收窄。

在 TypeScript 中，检查 `typeof` 返回的值是一种类型保护。
因为 TypeScript 编码了 `typeof` 如何对不同的值进行操作，所以它知道 JavaScript 中 `typeof` 的一些奇特之处。例如，注意到在上面的列表中，`typeof` 并不返回字符串 `null`。
让我们看一下下面的例子："

```ts twoslash
// @errors: 2531 18047
function printAll(strs: string | string[] | null) {
  if (typeof strs === "object") {
    for (const s of strs) {
      console.log(s);
    }
  } else if (typeof strs === "string") {
    console.log(strs);
  } else {
    // do nothing
  }
}
```


在 `printAll` 函数中，我们尝试检查 `strs` 是否是对象，以判定它是否是数组类型（现在可能是强调 JavaScript 中数组是对象类型的好时机）。但事实证明，在 JavaScript 中，`typeof null` 实际上是 `"object"`！这是历史的不幸意外。

有足够经验的用户可能不会感到惊讶，但并非所有人都在 JavaScript 中遇到过这个问题；幸运的是，TypeScript 会让我们知道 `strs` 只被收敛到 `string[] | null`，而不仅仅是 `string[]`。

这可能是一个很好的过渡，让我引入我们所称之为 "真值" 检查的内容。

# 真值收敛

"真值"（Truthiness）虽然不是你在字典中能找到的词，但它确实是你在 JavaScript 中经常会听到的一个概念。

在 JavaScript 中，我们可以在条件语句、`&&`、`||`、`if` 语句、布尔取反（`!`）等中使用任何表达式。
例如，`if` 语句的条件并不总是需要是 `boolean` 类型的。

```ts twoslash
function getUsersOnlineMessage(numUsersOnline: number) {
  if (numUsersOnline) {
    return `There are ${numUsersOnline} online now!`;
  }
  return "Nobody's here. :(";
}
```


在 JavaScript 中，诸如 `if` 这样的结构首先会将其条件“强制”转化为 `boolean` 类型以解析它们，然后根据结果是 `true` 还是 `false` 来选择执行哪个分支。
例如像这样的值：

- `0`
- `NaN`
- `""` (空字符串)
- `0n` (`bigint` 版本的 0)
- `null`
- `undefined`

全部会被强制转换为 `false` ，而其他值则会被强制转换为 `true`。
你可以通过运行 `Boolean` 函数，或者使用更简洁的双重布尔否定来将值强制转换为 `boolean`。 （后者的优点在于，TypeScript 会推断出一个更精确的字面量布尔类型 `true`，而对于第一种情况它则会推断为 `boolean` 类型。）"
all coerce to `false`, and other values get coerced to `true`.

```ts twoslash
// both of these result in 'true'
Boolean("hello"); // type: boolean, value: true
!!"world"; // type: true,    value: true
```

这种行为的利用相当普遍，尤其是用于防止出现 `null` 或 `undefined` 这样的值。
例如，让我们试着在我们的 `printAll` 函数中使用。

```ts twoslash
function printAll(strs: string | string[] | null) {
  if (strs && typeof strs === "object") {
    for (const s of strs) {
      console.log(s);
    }
  } else if (typeof strs === "string") {
    console.log(strs);
  }
}
```

您会注意到，我们通过检查 `strs` 是否为真消除了上述错误。
这至少可以防止我们在运行代码时出现可怕的错误，例如：

```txt
TypeError: null is not iterable
```

不过，请记住，对基本类型进行真值检查往往容易出错。
例如，考虑一下编写 `printAll` 函数的另一种尝试。

```ts twoslash {class: "do-not-do-this"}
function printAll(strs: string | string[] | null) {
  // !!!!!!!!!!!!!!!!
  //  不要这样做!
  //  继续读下去
  // !!!!!!!!!!!!!!!!
  if (strs) {
    if (typeof strs === "object") {
      for (const s of strs) {
        console.log(s);
      }
    } else if (typeof strs === "string") {
      console.log(strs);
    }
  }
}
```

我们将函数的整个主体都包在了一个真值性检查中，但这有一个微妙的缺点：我们可能不再正确处理空字符串的情况。

在这里 TypeScript 并没有对我们造成任何困扰，但如果你对 JavaScript 不是很熟悉，这种行为值得注意。 
TypeScript 可以经常帮助你在早期捕获 bug，但如果你选择对一个值 _视而不见_，那 TypeScript 能做的就有限了，也不可能过于喋喋不休。
如果你愿意，你可以通过使用一个 linter 来确保处理这样的情况。

关于通过真值性进行缩小范围的最后一点说明是，使用 `!` 进行的布尔否定可以从否定的分支中过滤出数据。

```ts twoslash
function multiplyAll(
  values: number[] | undefined,
  factor: number
): number[] | undefined {
  if (!values) {
    return values;
  } else {
    return values.map((x) => x * factor);
  }
}
```

## 相等收敛（Equality narrowing）

TypeScript 同样使用 `switch` 语句和像 `===`，`!==`，`==`，和 `!=` 这样的相等检查来进行类型收敛。
例如：

```ts twoslash
function example(x: string | number, y: string | boolean) {
  if (x === y) {
    // We can now call any 'string' method on 'x' or 'y'.
    x.toUpperCase();
    // ^?
    y.toLowerCase();
    // ^?
  } else {
    console.log(x);
    //          ^?
    console.log(y);
    //          ^?
  }
}
```

当我们在上述例子中检查 `x` 和 `y` 是否相等时，TypeScript 知道它们的类型也必须相等。
由于 `string` 是 `x` 和 `y` 唯一可能的共同类型，因此 TypeScript 知道在第一个分支中，`x` 和 `y` 必须是 `string` 类型。

同时针对特定的字面量值（而非变量）进行检查也同样可行。
在关于真值性缩小的章节中，我们编写了一个 `printAll` 函数，这个函数容易出错，因为它意外地没有正确处理空字符串。
相反，我们应该可以对 `null` 进行特定的检查，并且 TypeScript 仍会正确地从 `strs` 的类型中排除 `null`。"

```ts twoslash
function printAll(strs: string | string[] | null) {
  if (strs !== null) {
    if (typeof strs === "object") {
      for (const s of strs) {
        //            ^?
        console.log(s);
      }
    } else if (typeof strs === "string") {
      console.log(strs);
      //          ^?
    }
  }
}
```

JavaScript 的宽松相等检查，如 `==` 和 `!=`，也能够正确地进行缩小范围。
如果你不太熟悉，`== null` 的检查实际上不仅仅验证一个值是否特指 `null`，它还会检查值是否可能为 `undefined`（译者注：即 `null == undefined` 的值为 true）。
同理，`== undefined` 的检查也可以验证一个值是否为 `null` 或者 `undefined`。

```ts twoslash
interface Container {
  value: number | null | undefined;
}

function multiplyValue(container: Container, factor: number) {
  // Remove both 'null' and 'undefined' from the type.
  if (container.value != null) {
    console.log(container.value);
    //                    ^?

    // Now we can safely multiply 'container.value'.
    container.value *= factor;
  }
}
```

## 使用 `in` 运算符的类型收敛

JavaScript 有一个运算符可以确定一个对象或其原型链是否存在一个具有特定名称的属性：那就是 `in` 运算符。
TypeScript 将这一点视为收敛潜在类型的一种方法。

例如，对于代码：`"value" in x`，其中 `"value"` 是一个字符串字面量, `x` 是一个联合类型。
"true" 分支将 `x` 的类型收敛为那些具有可选或必需的 `value` 属性的类型，而 "false" 分支则将类型缩小为那些具有可选或缺失 `value` 属性的类型。

```ts twoslash
type Fish = { swim: () => void };
type Bird = { fly: () => void };

function move(animal: Fish | Bird) {
  if ("swim" in animal) {
    return animal.swim();
  }

  return animal.fly();
}
```

重申一下，在类型收敛过程中，可选属性将在两个分支中都存在。例如，人类既可以游泳也可以飞行（使用正确的设备），因此应该在 `in` 检查的两个分支中都出现：

<!-- prettier-ignore -->
```ts twoslash
type Fish = { swim: () => void };
type Bird = { fly: () => void };
type Human = { swim?: () => void; fly?: () => void };

function move(animal: Fish | Bird | Human) {
  if ("swim" in animal) {
    animal;
//  ^?
  } else {
    animal;
//  ^?
  }
}
```

## `instanceof` 类型收敛

JavaScript提供了一个运算符，用于检查一个值是否是另一个值的"实例"。
更具体地说，在JavaScript中，`x instanceof Foo` 会检查 `x` 的 _原型链_ 是否包含 `Foo.prototype`。
虽然我们不会在这里深入探讨，你会在我们讲解类时看到更多关于 `instanceof` 的用法，但它对于大多数可以用 `new` 构造的值来说仍然很有用。
你也许已经猜到了，`instanceof` 也是一个类型守卫，TypeScript 会在被 `instanceof` 守卫的分支中进行类型收敛。

```ts twoslash
function logValue(x: Date | string) {
  if (x instanceof Date) {
    console.log(x.toUTCString());
    //          ^?
  } else {
    console.log(x.toUpperCase());
    //          ^?
  }
}
```

## 赋值操作

如我们之前所提，当我们对任何变量进行赋值时，TypeScript 会查看赋值操作的右侧内容，并适当地收敛左侧变量的类型。

```ts twoslash
let x = Math.random() < 0.5 ? 10 : "hello world!";
//  ^?
x = 1;

console.log(x);
//          ^?
x = "goodbye!";

console.log(x);
//          ^?
```

注意，上述所有赋值都是有效的。
即使 `x` 在我们初次分配后其观察到的类型变为 `number`，我们仍能够将一个 `string` 赋值给 `x`。
这是因为 `x` 的 _声明类型_（ `x` 开始的类型）是 `string | number` ，赋值的可行性总是依据声明类型进行检查。

如果我们将一个 `boolean` 交给 `x`，我们会得到一个错误，因为这并不是声明类型的一部分。

```ts twoslash
// @errors: 2322
let x = Math.random() < 0.5 ? 10 : "hello world!";
//  ^?
x = 1;

console.log(x);
//          ^?
x = true;

console.log(x);
//          ^?
```

## 控制流分析

到目前为止，我们已经通过一些基本的示例讲解了 TypeScript 在特定分支中是如何进行类型收敛的。
但这不仅仅是从每个变量开始向上寻找 `if`，`while`，条件语句中的类型守卫那么简单。
比如: 

```ts twoslash
function padLeft(padding: number | string, input: string) {
  if (typeof padding === "number") {
    return " ".repeat(padding) + input;
  }
  return padding + input;
}
```

`padLeft` 在其第一个 `if` 块中就可能 `return`。
TypeScript 能够分析此代码，并看到在 `padding` 为 `number` 的情况下，函数体的其余部分(`return padding + input;`) 是 _无法到达的_。
因此，对于函数的剩余部分，它能够从 `padding` 的类型中去掉 `number`（从 `string | number` 收敛到 `string`）。

这种基于可达性的代码分析被称为 _控制流分析_，TypeScript 在遇到类型守卫和赋值时，会使用这种流程分析来收敛类型。
当分析一个变量时，控制流可以反复分裂和合并，每个点上的变量都可能有不同的类型。

```ts twoslash
function example() {
  let x: string | number | boolean;

  x = Math.random() < 0.5;

  console.log(x);
  //          ^?

  if (Math.random() < 0.5) {
    x = "hello";
    console.log(x);
    //          ^?
  } else {
    x = 100;
    console.log(x);
    //          ^?
  }

  return x;
  //     ^?
}
```

## 使用类型谓词

到目前为止，我们使用了现有的 JavaScript 结构来处理类型的收敛，然而，有时你可能希望更直接地控制代码中的类型如何变化。

要定义一个用户自定义的类型守卫，我们只需要定义一个返回类型为 _类型谓词_ 的函数：

```ts twoslash
type Fish = { swim: () => void };
type Bird = { fly: () => void };
declare function getSmallPet(): Fish | Bird;
// ---cut---
function isFish(pet: Fish | Bird): pet is Fish {
  return (pet as Fish).swim !== undefined;
}
```

在这个例子中，`pet is Fish` 是我们的类型谓词。
谓词的形式为 `parameterName is Type`，其中 `parameterName` 必须是当前函数签名中的一个参数的名称。
只要用某个变量调用 `isFish`，如果原始类型兼容，TypeScript 就会将该变量 _收敛_ 为特定类型。
`pet is Fish` is our type predicate in this example.

```ts twoslash
type Fish = { swim: () => void };
type Bird = { fly: () => void };
declare function getSmallPet(): Fish | Bird;
function isFish(pet: Fish | Bird): pet is Fish {
  return (pet as Fish).swim !== undefined;
}
// ---cut---
// Both calls to 'swim' and 'fly' are now okay.
let pet = getSmallPet();

if (isFish(pet)) {
  pet.swim();
} else {
  pet.fly();
}
```

注意到，TypeScript 不仅知道在 `if` 分支中 `pet` 是一个 `Fish`；
它还知道在 `else` 分支中，你_没有_ `Fish`，所以你一定有一个 `Bird`。

你可以使用类型守卫 `isFish` 来过滤一个 `Fish | Bird` 的数组并得到一个 `Fish` 的数组：

```ts twoslash
type Fish = { swim: () => void; name: string };
type Bird = { fly: () => void; name: string };
declare function getSmallPet(): Fish | Bird;
function isFish(pet: Fish | Bird): pet is Fish {
  return (pet as Fish).swim !== undefined;
}
// ---cut---
const zoo: (Fish | Bird)[] = [getSmallPet(), getSmallPet(), getSmallPet()];
const underWater1: Fish[] = zoo.filter(isFish);
// or, equivalently
const underWater2: Fish[] = zoo.filter(isFish) as Fish[];

// The predicate may need repeating for more complex examples
const underWater3: Fish[] = zoo.filter((pet): pet is Fish => {
  if (pet.name === "sharkey") return false;
  return isFish(pet);
});
```

此外，类可以 [使用 `this is Type`](/zh/docs/handbook/2/classes.html#this-based-type-guards) 来收敛它们的类型。

## 断言函数

类型也可以通过 [断言函数](/docs/handbook/release-notes/typescript-3-7.html#assertion-functions) 进行收敛。

# 判别联合

我们迄今为止看过的大部分示例都集中在缩小简单类型（如 `string`、`boolean` 和 `number`）的单个变量上。
虽然这很常见，但在 JavaScript 中，我们大多数时候会处理稍微复杂一些的结构。

以某些动机为例，让我们想象我们正在尝试编码如圆圈和正方形这样的形状。
圆圈处理它们的半径，正方形则处理它们的边长。
我们将使用一个名为 `kind` 的字段来表明我们正在处理哪种形状。
接下来，这是我们首次尝试定义 `Shape` 的方式。

```ts twoslash
interface Shape {
  kind: "circle" | "square";
  radius?: number;
  sideLength?: number;
}
```

注意，我们正在使用字符串字面量类型的联合：`·circle"` 和 `"square"` 来确定我们应该将形状视作圆形还是正方形。
通过使用 `"circle" | "square"` 而不是 `string`，我们可以避免拼写错误的问题。

```ts twoslash
// @errors: 2367
interface Shape {
  kind: "circle" | "square";
  radius?: number;
  sideLength?: number;
}

// ---cut---
function handleShape(shape: Shape) {
  // oops!
  if (shape.kind === "rect") {
    // ...
  }
}
```

我们可以编写一个 `getArea` 函数，根据处理的是圆形还是正方形应用适当的逻辑。
我们先尝试处理圆形。

```ts twoslash
// @errors: 2532 18048
interface Shape {
  kind: "circle" | "square";
  radius?: number;
  sideLength?: number;
}

// ---cut---
function getArea(shape: Shape) {
  return Math.PI * shape.radius ** 2;
}
```

<!-- TODO -->

在 [`strictNullChecks`](/zh/tsconfig#strictNullChecks) 下，这会给我们一个错误 - 这是合理的，因为 `radius` 可能未被定义。
但是，如果我们对 kind 属性进行适当的检查呢？

```ts twoslash
// @errors: 2532 18048
interface Shape {
  kind: "circle" | "square";
  radius?: number;
  sideLength?: number;
}

// ---cut---
function getArea(shape: Shape) {
  if (shape.kind === "circle") {
    return Math.PI * shape.radius ** 2;
  }
}
```

嗯，TypeScript 还是不知道这里应该怎么做。
我们遇到了一个情况，我们对我们的值的了解超过了类型检查器。
我们可以尝试使用非空断言（在 `shape.radius` 后加一个 `!`），来表明 `radius` 肯定存在。

```ts twoslash
interface Shape {
  kind: "circle" | "square";
  radius?: number;
  sideLength?: number;
}

// ---cut---
function getArea(shape: Shape) {
  if (shape.kind === "circle") {
    return Math.PI * shape.radius! ** 2;
  }
}
```

但这并不理想。
我们必须大声地向类型检查器提出非空断言(`!`)才能让它相信 `shape.radius` 已定义，但如果我们开始移动代码，这些断言容易出错。
另外，在 [`strictNullChecks`](/zh/tsconfig#strictNullChecks) 之外，我们仍然可以偶然访问到这些字段（因为在读取时，可选属性总是被假定为存在）。
我们肯定可以做得更好。

编码 `Shape` 的问题在于，类型检查器无法根据 `kind` 属性知道 `radius` 或 `sideLength` 是否存在。
我们需要将我们自己的认知传达给类型检查器。
牢记这一点，让我们再试一次定义 `Shape`。

```ts twoslash
interface Circle {
  kind: "circle";
  radius: number;
}

interface Square {
  kind: "square";
  sideLength: number;
}

type Shape = Circle | Square;
```

这里，我们已经合理地将 `Shape` 分离成两种不同的类型，它们的 `kind` 属性具有不同的值，但在各自的类型中， `radius` 和 `sideLength` 被声明为必需的属性。

让我们来看看尝试访问 `Shape` 的 `radius` 时会发生什么。"

```ts twoslash
// @errors: 2339
interface Circle {
  kind: "circle";
  radius: number;
}

interface Square {
  kind: "square";
  sideLength: number;
}

type Shape = Circle | Square;

// ---cut---
function getArea(shape: Shape) {
  return Math.PI * shape.radius ** 2;
}
```

就像我们对 `Shape` 的第一个定义一样，这依旧是一个错误。
当 `radius` 是可选的时候，我们得到了一个错误（启用 [`strictNullChecks`](/zh/tsconfig#strictNullChecks)），因为 TypeScript 无法确定属性是否存在。
现在 `Shape` 是一个联合，TypeScript 告诉我们 `shape` 可能是一个 `Square`，而 `Square` 上没有定义 `radius`！
两种解释都是正确的，但只有联合编码的 `Shape` 会导致无论如何配置 [`strictNullChecks`](/zh/tsconfig#strictNullChecks) 都会引发错误。

但是如果我们再次尝试检查 `kind` 属性会怎么样呢？

```ts twoslash
interface Circle {
  kind: "circle";
  radius: number;
}

interface Square {
  kind: "square";
  sideLength: number;
}

type Shape = Circle | Square;

// ---cut---
function getArea(shape: Shape) {
  if (shape.kind === "circle") {
    return Math.PI * shape.radius ** 2;
    //               ^?
  }
}
```

这消除了错误！
当联合中的每一种类型都包含有字面类型的公共属性时，TypeScript 将其认为是一个 _判别联合_ ，并可以收敛联合的成员。

在这种情况下，`kind` 就是那个公共属性（这就是被认为是 `Shape` 的 _判别属性_）。
检查 `kind` 属性是否为 `"circle"` 消除了在 `Shape` 中所有没有 `kind` 属性类型为 `"circle"` 的类型。
这将 `shape` 缩小到了 `Circle` 类型。

同样的检查也适用于 `switch` 语句。
现在我们可以尝试写我们完整的 `getArea`，而无需任何烦人的 `!` 非空断言。

```ts twoslash
interface Circle {
  kind: "circle";
  radius: number;
}

interface Square {
  kind: "square";
  sideLength: number;
}

type Shape = Circle | Square;

// ---cut---
function getArea(shape: Shape) {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    //                 ^?
    case "square":
      return shape.sideLength ** 2;
    //       ^?
  }
}
```

这里重要的事情是针对 `Shape` 进行编码。
将正确的信息告知 TypeScript -- 即 `Circle` 和 `Square` 实际上是两种具有特定 `kind` 字段的不同类型 -- 这一步是非常关键的。
这样做可以让我们编写的 TypeScript 代码看起来和我们原本要写的 JavaScript 没有区别。
从那里开始，类型系统能够做出 "正确" 的事情，并找出我们 `switch` 语句中每个分支的类型。

> 另外，尝试着玩一玩上述例子并删除一些 return 关键字。
> 你会看到，类型检查可以帮助我们避免在 `switch` 语句中意外跳出不同子句导致的错误。

判别联合不仅仅在谈论圆形和正方形时有用。
它们适合表示 JavaScript 中的任何类型的消息方案，比如在网络上发送消息（客户端/服务器通信），或者在状态管理框架中进行编码变化。

# `never` 类型

在进行类型收敛时，你可能将联合体的选项减少到没有任何可能性，什么都没有剩下。
在这些情况下，TypeScript 会使用 `never` 类型来表示一个不应存在的状态。

# 穷尽性检查

`never` 类型可以赋给任何类型；然而，没有任何类型可以赋给 `never`（除了 `never` 本身）。这意味着你可以使用类型收敛，并依赖 `never` 出现来在 `switch` 语句中进行穷尽性检查。

例如，为我们的 `getArea` 函数添加一个试图将形状赋值给 `never` 的 `default`，当每一种可能的情况都得到处理时，将不会抛出错误。

```ts twoslash
interface Circle {
  kind: "circle";
  radius: number;
}

interface Square {
  kind: "square";
  sideLength: number;
}
// ---cut---
type Shape = Circle | Square;

function getArea(shape: Shape) {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "square":
      return shape.sideLength ** 2;
    default:
      const _exhaustiveCheck: never = shape;
      return _exhaustiveCheck;
  }
}
```

将一个新的成员添加到 Shape 联合类型中，将会引发 TypeScript 错误：

```ts twoslash
// @errors: 2322
interface Circle {
  kind: "circle";
  radius: number;
}

interface Square {
  kind: "square";
  sideLength: number;
}
// ---cut---
interface Triangle {
  kind: "triangle";
  sideLength: number;
}

type Shape = Circle | Square | Triangle;

function getArea(shape: Shape) {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "square":
      return shape.sideLength ** 2;
    default:
      const _exhaustiveCheck: never = shape;
      return _exhaustiveCheck;
  }
}
```
