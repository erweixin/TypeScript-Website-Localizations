---
title: 有关函数的更多信息
layout: docs
permalink: /zh/docs/handbook/2/functions.html
oneline: "了解 TypeScript 中函数的工作原理。"
---

函数是任何应用程序的基本构建块，无论它们是本地函数、从其他模块导入的函数，还是类上的方法。
它们也是值，就像其他值一样，TypeScript 有许多方式来描述函数如何被调用。
让我们学习如何编写描述函数的类型。

## 函数类型表达式

描述函数的最简单方式是使用 _函数类型表达式_。
这些类型在语法上类似于箭头函数

```ts twoslash
function greeter(fn: (a: string) => void) {
  fn("Hello, World");
}

function printToConsole(s: string) {
  console.log(s);
}

greeter(printToConsole);
```
语法 `(a: string) => void` 表示“一个函数，有一个名为 `a` 的参数，类型为 `string`，没有返回值”。
就像函数声明一样，如果参数类型没有指定，它默认为 `any`。

> 注意参数名称是**必需的**。函数类型 `(string) => void` 表示“一个函数，有一个名为 `string` 的参数，类型为 `any`”。

当然，我们可以使用类型别名来命名函数类型：

```ts twoslash
type GreetFunction = (a: string) => void;
function greeter(fn: GreetFunction) {
  // ...
}
```


## 调用签名

在 JavaScript 中，函数除了可调用外，还可以有属性。
然而，函数类型表达式语法不允许声明属性。
如果我们想描述一个具有属性的可调用对象，我们可以在对象类型中编写一个 _调用签名_：

```ts twoslash
type DescribableFunction = {
  description: string;
  (someArg: number): boolean;
};
function doSomething(fn: DescribableFunction) {
  console.log(fn.description + " returned " + fn(6));
}

function myFunc(someArg: number) {
  return someArg > 3;
}
myFunc.description = "default description";

doSomething(myFunc);
```

注意，与函数类型表达式相比，语法稍有不同 —— 在参数列表和返回类型之间使用 `:` 而不是 `=>`。

## 构造签名

JavaScript 函数也可以使用 `new` 运算符调用。
TypeScript 将这些称为 _构造函数_，
因为它们通常会创建一个新对象。你可以通过在调用签名前添加 `new` 关键字来编写一个 _构造签名_：

```ts twoslash
type SomeObject = any;
// ---cut---
type SomeConstructor = {
  new (s: string): SomeObject;
};
function fn(ctor: SomeConstructor) {
  return new ctor("hello");
}
```

有些对象，如 JavaScript 的 `Date` 对象，可以带或不带 `new` 调用。
你可以在同一类型中任意组合调用和构造签名：

```ts twoslash
interface CallOrConstruct {
  (n?: number): string;
  new (s: string): Date;
}
```

## 泛型函数

编写函数时，输入的类型通常与输出的类型相关，或者两个输入的类型以某种方式相关。
让我们考虑一个返回数组第一个元素的函数：

```ts twoslash
function firstElement(arr: any[]) {
  return arr[0];
}
```

这个函数完成了它的工作，但不太好的是返回类型是 `any`。
如果函数返回数组元素的类型会更好。

 TypeScript 中，当我们想描述两个值之间的对应关系时，可以使用 _泛型_。
 我们通过在函数签名中声明一个 _类型参数_ 来实现这一点：
 
 ```ts twoslash
function firstElement<Type>(arr: Type[]): Type | undefined {
  return arr[0];
}
```

通过在这个函数中添加类型参数 `Type` 并在两个地方使用它，我们在函数的输入（数组）和输出（返回值）之间创建了一个链接。
现在当我们调用它时，会输出更具体的类型：

```ts twoslash
declare function firstElement<Type>(arr: Type[]): Type | undefined;
// ---cut---
// s is of type 'string'
const s = firstElement(["a", "b", "c"]);
// n is of type 'number'
const n = firstElement([1, 2, 3]);
// u is of type undefined
const u = firstElement([]);
```

### 推断（Inference）

注意我们在这个示例中没有指定 `Type`。
类型是由 TypeScript 自动选择的 —— 自动推断的。

我们也可以使用多个类型参数。
例如，`map` 的独立版本看起来像这样：

```ts twoslash
// prettier-ignore
function map<Input, Output>(arr: Input[], func: (arg: Input) => Output): Output[] {
  return arr.map(func);
}
```

在这个例子中，TypeScript 能够推断出 `Input` 类型参数（从给定的 `string` 数组），以及基于函数表达式的返回值（`number`）的 `Output` 类型参数。

### 约束（Constraints）

我们已经编写了一些通用的函数，它们可以在 _任何_ 类型的值上工作。
有时，我们想要关联两个值，但只能针对特定的值子集进行操作。
这种情况下，我们可以使用一个 _约束_ 来限制类型参数可以接受的类型。

让我们编写一个函数，返回两个值中较长的一个。
为此，我们需要一个属性 `length` ，并且它是一个数字。
我们通过编写一个 `extends` 子句来 _约束_ 类型参数：

```ts twoslash
// @errors: 2345 2322
function longest<Type extends { length: number }>(a: Type, b: Type) {
  if (a.length >= b.length) {
    return a;
  } else {
    return b;
  }
}

// longerArray is of type 'number[]'
const longerArray = longest([1, 2], [1, 2, 3]);
// longerString is of type 'alice' | 'bob'
const longerString = longest("alice", "bob");
// Error! Numbers don't have a 'length' property
const notOK = longest(10, 100);
```

在这个例子中有一些有趣的地方值得注意。
我们让 TypeScript 来 _推断_ `longest` 的返回类型。
返回类型推断也可以应用于泛型函数。

由于我们将 `Type` 约束为 `{ length: number }`，我们可以访问参数 `a` 和 `b` 的 `.length` 属性。
如果没有类型约束，我们将无法访问这些属性，因为值可能是没有 length 属性的其他类型。

`longerArray` 和 `longerString` 的类型基于参数来推断。
记住，泛型就是关于用相同的类型关联两个或更多的值！

最后，正如我们所期望的，调用 `longest(10, 100)` 会被拒绝，因为 `number` 类型没有 `.length` 属性。

### 使用受约束的值时的常见错误

在使用泛型约束时，经常会遇到一些错误。

```ts twoslash
// @errors: 2322
function minimumLength<Type extends { length: number }>(
  obj: Type,
  minimum: number
): Type {
  if (obj.length >= minimum) {
    return obj;
  } else {
    return { length: minimum };
  }
}
```

这个函数看起来似乎是没问题的 - `Type` 被约束为 `{ length: number }` ，并且函数要么返回 `Type`，要么返回满足这个约束的值。
问题在于，这个函数承诺返回与传入的同种对象，而不仅仅是满足约束的某个对象。
如果这段代码合法，你可能会写出有错误的代码：

```ts twoslash
declare function minimumLength<Type extends { length: number }>(
  obj: Type,
  minimum: number
): Type;
// ---cut---
// 'arr' gets value { length: 6 }
const arr = minimumLength([1, 2, 3], 6);
// and crashes here because arrays have
// a 'slice' method, but not the returned object!
console.log(arr.slice(0));
```

### 指定类型参数

TypeScript 通常可以推断出泛型调用中预期的类型参数，但不总是如此。
例如，假设你编写了一个函数来合并两个数组：

```ts twoslash
function combine<Type>(arr1: Type[], arr2: Type[]): Type[] {
  return arr1.concat(arr2);
}
```

通常，如果用不匹配的数组调用这个函数，会出错：

```ts twoslash
// @errors: 2322
declare function combine<Type>(arr1: Type[], arr2: Type[]): Type[];
// ---cut---
const arr = combine([1, 2, 3], ["hello"]);
```

但是，如果你打算这样做，可以手动指定 `Type`：

```ts twoslash
declare function combine<Type>(arr1: Type[], arr2: Type[]): Type[];
// ---cut---
const arr = combine<string | number>([1, 2, 3], ["hello"]);
```

### 编写良好泛型函数的指南

编写通用函数很有趣，但有时我们可能会对类型参数过于痴迷。
过多的类型参数或在不需要时使用约束，可能会降低类型推断的成功率，从而使调用你的函数的人感到困扰。

#### 减少类型参数的使用

这里有两种看起来相似的编写函数的方法：

```ts twoslash
function firstElement1<Type>(arr: Type[]) {
  return arr[0];
}

function firstElement2<Type extends any[]>(arr: Type) {
  return arr[0];
}

// a: number (good)
const a = firstElement1([1, 2, 3]);
// b: any (bad)
const b = firstElement2([1, 2, 3]);
```

乍一看，`firstElement1` 和 `firstElement2` 可能看起来相同，但 `firstElement1` 是编写此函数的更好方式。
它推断的返回类型是 `Type`，但 `firstElement2` 的推断返回类型是 `any`，这是因为 TypeScript 必须使用约束类型来解析 `arr[0]` 表达式，而不是"等待"调用过程中来解析元素。

> **规则**：尽可能使用类型参数本身，而不是对它进行约束。

#### 使用更少的类型参数

这里是一对类似的函数：

```ts twoslash
function filter1<Type>(arr: Type[], func: (arg: Type) => boolean): Type[] {
  return arr.filter(func);
}

function filter2<Type, Func extends (arg: Type) => boolean>(
  arr: Type[],
  func: Func
): Type[] {
  return arr.filter(func);
}
```

我们创建了一个类型参数 `Func`，这个参数 并没有关联两个值 。
这总是一个警告信号，因为这意味着想要指定类型参数的调用者必须手动为无关紧要的原因额外指定一个类型参数。
Func 除了让函数更难阅读和推理外，没有做任何事情！

> **规则**: 总是尽可能少使用类型参数

#### 多次使用才应该考虑增加类型参数

有时我们可能会忘记一个函数可能不需要泛型：

```ts twoslash
function greet<Str extends string>(s: Str) {
  console.log("Hello, " + s);
}

greet("world");
```

我们完全可以写出一个更简洁的版本：

```ts twoslash
function greet(s: string) {
  console.log("Hello, " + s);
}
```

请记住，类型参数用于 _关联多个值的类型_。
如果类型参数在函数签名中只使用一次，那么它其实并未关联任何类型。
这也包括推断的返回类型；例如，如果 `Str` 是 `greet` 的推断返回类型的一部分，那么它就是在关联参数类型和返回类型，尽管在编写的代码中只出现了一次，但实际上它被使用了 _两次_。

> **规则**: 如果类型参数只在一个地方出现，那么强烈建议你重新考虑是否真的需要它。

## 可选参数

JavaScript 中的函数经常需要接收可变数量的参数。
例如，`number` 的 `toFixed` 方法接收一个可选的数字计数：

```ts twoslash
function f(n: number) {
  console.log(n.toFixed()); // 0 arguments
  console.log(n.toFixed(3)); // 1 argument
}
```

我们可以通过在 TypeScript 中用 `?` 标记参数为 _可选_ 来模拟这种情况：

```ts twoslash
function f(x?: number) {
  // ...
}
f(); // OK
f(10); // OK
```

虽然参数被指定为 `number` 类型，但 `x` 参数实际上将具有 `number | undefined` 类型，因为在 JavaScript 中未指定的参数会被赋值为 `undefined`。

你还可以提供参数的 _默认值_ ：

```ts twoslash
function f(x = 10) {
  // ...
}
```

现在在 `f` 的函数体中，`x` 将具有 `number` 类型，因为任何 `undefined` 参数都将被 10 替换。
请注意，当参数是可选的时候，调用者总是可以传递 `undefined`，因为这只是在模拟 "缺失" 的参数：

```ts twoslash
declare function f(x?: number): void;
// ---cut---
// All OK
f();
f(10);
f(undefined);
```

### Optional Parameters in Callbacks

Once you've learned about optional parameters and function type expressions, it's very easy to make the following mistakes when writing functions that invoke callbacks:

```ts twoslash
function myForEach(arr: any[], callback: (arg: any, index?: number) => void) {
  for (let i = 0; i < arr.length; i++) {
    callback(arr[i], i);
  }
}
```

What people usually intend when writing `index?` as an optional parameter is that they want both of these calls to be legal:

```ts twoslash
// @errors: 2532 18048
declare function myForEach(
  arr: any[],
  callback: (arg: any, index?: number) => void
): void;
// ---cut---
myForEach([1, 2, 3], (a) => console.log(a));
myForEach([1, 2, 3], (a, i) => console.log(a, i));
```

What this _actually_ means is that _`callback` might get invoked with one argument_.
In other words, the function definition says that the implementation might look like this:

```ts twoslash
// @errors: 2532 18048
function myForEach(arr: any[], callback: (arg: any, index?: number) => void) {
  for (let i = 0; i < arr.length; i++) {
    // I don't feel like providing the index today
    callback(arr[i]);
  }
}
```

In turn, TypeScript will enforce this meaning and issue errors that aren't really possible:

<!-- prettier-ignore -->
```ts twoslash
// @errors: 2532 18048
declare function myForEach(
  arr: any[],
  callback: (arg: any, index?: number) => void
): void;
// ---cut---
myForEach([1, 2, 3], (a, i) => {
  console.log(i.toFixed());
});
```

In JavaScript, if you call a function with more arguments than there are parameters, the extra arguments are simply ignored.
TypeScript behaves the same way.
Functions with fewer parameters (of the same types) can always take the place of functions with more parameters.

> **Rule**: When writing a function type for a callback, _never_ write an optional parameter unless you intend to _call_ the function without passing that argument

## 函数重载

一些 JavaScript 函数可以接受不同数量和类型的参数调用。
例如，你可能要编写一个生成 `Date` 的函数，它接收的参数既可以是时间戳（一个参数），也可以是年/月/日规定（三个参数）。

在 TypeScript 中，我们可以通过编写 _重载签名_ 来指定可以以不同方式调用的函数。
为此，我们需要书写一些函数签名（通常两个或更多个），然后再写出函数体：

```ts twoslash
// @errors: 2575
function makeDate(timestamp: number): Date;
function makeDate(m: number, d: number, y: number): Date;
function makeDate(mOrTimestamp: number, d?: number, y?: number): Date {
  if (d !== undefined && y !== undefined) {
    return new Date(y, mOrTimestamp, d);
  } else {
    return new Date(mOrTimestamp);
  }
}
const d1 = makeDate(12345678);
const d2 = makeDate(5, 5, 5);
const d3 = makeDate(1, 3);
```

在这个例子中，我们定义了两个重载：一个接受一参数，另一个接受三个参数。
这前两个签名就是我们所说的 _重载签名_。

然后，我们编写了一个与之兼容的函数实现。
函数具有 _实现_ 签名，但这个签名不能直接被调用。
尽管我们写出了一个在一个必需参数后有两个可选参数的函数，但它不能被两个参数来调用！

### 重载签名和实现签名（译者注：实现签名指的是实现功能的函数体，例如上面代码的第三个 markData 那一行）

这是一个常见的困扰来源。
经常有人会写下这样的代码，然而他们会不明白为什么会有错误：

```ts twoslash
// @errors: 2554
function fn(x: string): void;
function fn() {
  // ...
}
// Expected to be able to call with zero arguments
fn();
```

再次强调，编写函数体时使用的签名在外部是"看不见"的。 

> _实现签名_ 从外部是不可见的。
> 当编写一个重载函数时，你应该在函数的实现上方总应该有 _两个_ 或更多的签名。

实现签名也必须与重载签名兼容。
例如，以下的函数有错误，因为实现签名与重载没有正确的匹配：

```ts twoslash
// @errors: 2394
function fn(x: boolean): void;
// Argument type isn't right
function fn(x: string): void;
function fn(x: boolean) {}
```

```ts twoslash
// @errors: 2394
function fn(x: string): string;
// Return type isn't right
function fn(x: number): boolean;
function fn(x: string | number) {
  return "oops";
}
```

### 编写良好的重载

像泛型一样，你在使用函数重载时应该遵循一些指南。
遵循这些原则将使你的函数更容易调用，更容易理解，更容易实现。

考虑一个函数，它返回字符串或数组的长度：

```ts twoslash
function len(s: string): number;
function len(arr: any[]): number;
function len(x: any) {
  return x.length;
}
```

此函数是没有问题的；我们可以用字符串或数组来调用它。
然而，我们不能使用可能是字符串或数组的值来调用它，因为TypeScript只能将函数调用解析到一个重载：

```ts twoslash
// @errors: 2769
declare function len(s: string): number;
declare function len(arr: any[]): number;
// ---cut---
len(""); // OK
len([0]); // OK
len(Math.random() > 0.5 ? "hello" : [0]);
```

因为所有的重载都具有相同的参数数量和返回类型，我们可以改写为一个没有重载的版本的函数：

```ts twoslash
function len(x: any[] | string) {
  return x.length;
}
```

这样就好多了！
调用者可以使用任意一种类型的值来调用这个函数，作为附加福利，我们不需要找出一个正确的实现签名。

> 尽可能选择使用联合类型的参数，而不是重载

## 声明函数中的 `this`

TypeScript会通过代码流分析推断函数中 `this` 的类型，例如下面的代码：

```ts twoslash
const user = {
  id: 123,

  admin: false,
  becomeAdmin: function () {
    this.admin = true;
  },
};
```

TypeScript 明白 `user.becomeAdmin` 函数中的 `this` 对应于外层的对象 `user`。在许多情况下，这已经足够了，但也存在许多你需要更多地控制 `this` 代表什么对象的情况。JavaScript 规范规定你不能使用 `this` 作为一个参数名，所以 TypeScript 利用这个语法空间让你在函数体中声明 `this` 的类型。

```ts twoslash
interface User {
  id: number;
  admin: boolean;
}
declare const getDB: () => DB;
// ---cut---
interface DB {
  filterUsers(filter: (this: User) => boolean): User[];
}

const db = getDB();
const admins = db.filterUsers(function (this: User) {
  return this.admin;
});
```

这种模式在回调风格的 APIs 中很常见，通常有另一个对象控制你的函数何时被调用。注意，你需要使用 `function`，而不是箭头函数，来获取这种行为：

```ts twoslash
// @errors: 7041 7017
interface User {
  id: number;
  isAdmin: boolean;
}
declare const getDB: () => DB;
// ---cut---
interface DB {
  filterUsers(filter: (this: User) => boolean): User[];
}

const db = getDB();
const admins = db.filterUsers(() => this.admin);
```

## 需要了解的其他类型

在处理函数类型时，你要注意一些额外的类型。
虽然你可以在任何地方使用这些类型，但它们在函数的上下文中特别相关。

### `void`

`void` 表示不返回值的函数的返回值类型。
每当一个函数没有任何 return 语句，或者没有从这些返回语句中返回任何明确的值时，会推断出 void 类型：

```ts twoslash
// The inferred return type is void
function noop() {
  return;
}
```

在 JavaScript 中，一个不返回任何值的函数会隐式返回 undefined 值。
然而，在 TypeScript 中，void 和 undefined 并不是同样的东西。
本章后面的部分将详细介绍。

> `void` 不同于 `undefined`。

### `object`

特殊的 `object` 类型指代任何非原始值（`string`、`number`、`bigint`、`boolean`、`symbol`、`null` 或 `undefined`）。
这与 _空对象类型_ `{ }` 不同，也与全局类型 `Object` 不同。
你很可能永远不会用到 `Object`。

> `object` 不是 `Object`. **一直** 使用 `object`!

注意，在 JavaScript 中，函数值是对象：它们具有属性，它们的原型链中有 `Object.prototype`，是 Object 的实例(`instanceof Object`)，你可以在它们上调用 `Object.keys`，等等。
因此，函数类型在 TypeScript 中被视为 object。

### `unknown`

`unknown` 类型代表 _任何_ 值。
这和 `any` 类型相似，但 `unknown` 更安全，因为对 `unknown` 类型的值进行任何操作都是不合法的：

```ts twoslash
// @errors: 2571 18046
function f1(a: any) {
  a.b(); // OK
}
function f2(a: unknown) {
  a.b();
}
```

这在描述函数类型时非常有用，因为你可以描述接收任何值的函数，而函数体中不必包含 `any `值。

相反，你也可以描述返回 `unknown` 类型值的函数：

```ts twoslash
declare const someRandomString: string;
// ---cut---
function safeParse(s: string): unknown {
  return JSON.parse(s);
}

// Need to be careful with 'obj'!
const obj = safeParse(someRandomString);
```

### `never`

有些函数 _从不_ 返回任何值：

```ts twoslash
function fail(msg: string): never {
  throw new Error(msg);
}
```

never 类型代表 _从不_ 观察到的值。
在返回类型中，它意味着函数抛出了异常，或者结束了程序的执行。

当 TypeScript 判断联合类型中没有剩余的类型时，也会出现 `never` 类型。

```ts twoslash
function fn(x: string | number) {
  if (typeof x === "string") {
    // do something
  } else if (typeof x === "number") {
    // do something else
  } else {
    x; // has type 'never'!
  }
}
```

### `Function`

全局类型 `Function` 描述了所有 JavaScript 函数值上的属性，如 `bind`、`call`、`apply` 等。
它还有一个特殊的属性，即类型为 `Function` 的值总是可以被调用；这些调用会返回 `any`：

```ts twoslash
function doSomething(f: Function) {
  return f(1, 2, 3);
}
```

这是一个 _无类型函数调用_，通常最好避免使用，因为其返回类型为不安全的 `any`。

如果你需要接收任意函数，但不打算调用它，那么类型 `() => void` 通常更安全。

## 剩余参数和实参

<blockquote class='bg-reading'>
   <p>Background Reading:<br />
   <a href='https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/rest_parameters'>Rest Parameters</a><br/>
   <a href='https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax'>Spread Syntax</a><br/>
   </p>
</blockquote>

### 剩余参数

除了利用可选参数或重载来实现可以接受多种固定参数数量的函数外，我们也可以使用 _剩余参数_ 来定义接受 _无限制_ 数量的参数的函数。

剩余参数出现在所有其他参数之后，并使用 `...` 语法：

```ts twoslash
function multiply(n: number, ...m: number[]) {
  return m.map((x) => n * x);
}
// 'a' gets value [10, 20, 30, 40]
const a = multiply(10, 1, 2, 3, 4);
```

在 TypeScript 中，这些参数的类型注解隐含地为 `any[]` 而不是 `any`，并且给出的任何类型注解必须为 `Array<T>` 或 `T[]` 的形式，或者是元组类型（我们稍后会学习）。

### 剩余实参

相反地，我们可以使用扩展语法从可迭代对象（例如数组）中 _提供_ 可变数量的实参。
例如，数组的 `push` 方法接受任意数量的实参：

```ts twoslash
const arr1 = [1, 2, 3];
const arr2 = [4, 5, 6];
arr1.push(...arr2);
```

注意，在一般情况下，TypeScript 不会假设数组是不可变的。
这可能导致一些出人意料的行为：

```ts twoslash
// @errors: 2556
// Inferred type is number[] -- "an array with zero or more numbers",
// not specifically two numbers
const args = [8, 5];
const angle = Math.atan2(...args);
```

对于这种情况的最佳修复方法在一定程度上依赖于您的代码，但通常在 `const` 上下文中是最直接的解决方案：

```ts twoslash
// Inferred as 2-length tuple
const args = [8, 5] as const;
// OK
const angle = Math.atan2(...args);
```

使用剩余实参可能需要在目标旧的运行时环境时，打开 [`downlevelIteration`](/tsconfig#downlevelIteration) 选项。

<!-- TODO link to downlevel iteration -->

## 参数解构

<blockquote class='bg-reading'>
   <p>背景阅读:<br />
   <a href='https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment'>解构赋值</a><br/>
   </p>
</blockquote>

你可以使用参数解构来方便地将作为参数提供的对象解压缩到函数体中的一个或多个局部变量。
在 JavaScript 中，它看起来像这样：

```js
function sum({ a, b, c }) {
  console.log(a + b + c);
}
sum({ a: 10, b: 3, c: 9 });
```

对象的类型注解放在解构语法后面：

```ts twoslash
function sum({ a, b, c }: { a: number; b: number; c: number }) {
  console.log(a + b + c);
}
```

这可能看起来有点冗长，但你也可以在这里使用命名类型：

```ts twoslash
// Same as prior example
type ABC = { a: number; b: number; c: number };
function sum({ a, b, c }: ABC) {
  console.log(a + b + c);
}
```

## 函数的可分配性

### 返回类型 `void`

对于函数的 `void` 返回类型可能会产生一些不寻常，但预期的行为。

上下文类型化具有 `void` 返回类型的函数并不强制函数不返回某些内容。换句话说，带有 `void` 返回类型的上下文函数类型（`type voidFunc = () => void`）在实现时，可以返回 _任何_ 其他值，但这些值会被忽略。

因此，以下类型 `() => void` 的实现是有效的:

```ts twoslash
type voidFunc = () => void;

const f1: voidFunc = () => {
  return true;
};

const f2: voidFunc = () => true;

const f3: voidFunc = function () {
  return true;
};
```

当这些函数的返回值被分配给另一个变量时，它将保留 `void` 类型。

```ts twoslash
type voidFunc = () => void;

const f1: voidFunc = () => {
  return true;
};

const f2: voidFunc = () => true;

const f3: voidFunc = function () {
  return true;
};
// ---cut---
const v1 = f1();

const v2 = f2();

const v3 = f3();
```

这种行为的存在是为了确保即使 `Array.prototype.push` 返回一个数字，而 `Array.prototype.forEach` 方法期望一个返回类型为 `void` 的函数，以下代码仍然有效。

```ts twoslash
const src = [1, 2, 3];
const dst = [0];

src.forEach((el) => dst.push(el));
```

还有一个特殊情况需要注意，当一个字面函数定义具有 `void` 返回类型时，那个函数不得返回任何东西。

```ts twoslash
function f2(): void {
  // @ts-expect-error
  return true;
}

const f3 = function (): void {
  // @ts-expect-error
  return true;
};
```

如需了解更多关于 `void` 的信息，请参阅以下其他文档条目：

- [v2 handbook](https://www.doc.tslang.org/zh/docs/handbook/2/functions.html#void)
- [FAQ - "Why are functions returning non-void assignable to function returning void?"](https://github.com/Microsoft/TypeScript/wiki/FAQ#why-are-functions-returning-non-void-assignable-to-function-returning-void)
