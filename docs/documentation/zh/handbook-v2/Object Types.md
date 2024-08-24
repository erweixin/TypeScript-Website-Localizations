---
title: 对象类型
layout: docs
permalink: /zh/docs/handbook/2/objects.html
oneline: "TypeScript 如何描述 JavaScript 对象的结构。"
---

在JavaScript中，我们组织和传递数据的基本方式是通过对象。
在TypeScript中，我们通过 _对象类型_ 来表示这些数据。

正如我们所见，它们可以是匿名的：

```ts twoslash
function greet(person: { name: string; age: number }) {
  //                   ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  return "Hello " + person.name;
}
```

或者通过使用接口( interface )来命名：

```ts twoslash
interface Person {
  //      ^^^^^^
  name: string;
  age: number;
}

function greet(person: Person) {
  return "Hello " + person.name;
}
```

或者类型别称：

```ts twoslash
type Person = {
  // ^^^^^^
  name: string;
  age: number;
};

function greet(person: Person) {
  return "Hello " + person.name;
}
```

在上述三个例子中，我们都写了一些函数，这些函数接收包含属性 `name`（必须为 `string` 类型）和 `age`（必须为 `number` 类型）的对象。

## 快速参考

我们为 [`type` 和 `interface`](https://www.doc.tslang.org/zh/cheatsheets) 提供了速查表，如果你想快速查看每天常用的重要语法，可以参考。

## 属性修饰符

对象类型中的每个属性可以指定几件事：类型，属性是否可选，以及是否可以写入该属性。

### 可选属性

很多时候，我们会发现自己正在处理可能设置了属性的对象。
在这些情况下，我们可以通过在它们的名字后面添加一个问号（`?`）来将这些属性标记为 _可选_。

```ts twoslash
interface Shape {}
declare function getShape(): Shape;

// ---cut---
interface PaintOptions {
  shape: Shape;
  xPos?: number;
  //  ^
  yPos?: number;
  //  ^
}

function paintShape(opts: PaintOptions) {
  // ...
}

const shape = getShape();
paintShape({ shape });
paintShape({ shape, xPos: 100 });
paintShape({ shape, yPos: 100 });
paintShape({ shape, xPos: 100, yPos: 100 });
```

在这个例子中，`xPos` 和 `yPos` 都被视为可选的。
我们可以选择提供其中任何一个，因此上述对于 `paintShape` 的每一次调用都是有效的。
所谓的可选性，实质上就是如果属性被设置了，它就应该有一个特定的类型。

我们也可以读取那些属性——但是当我们在 [`strictNullChecks`](/zh/tsconfig#strictNullChecks) （严格空值检查）下进行读取时，TypeScript 会告诉我们它们可能 `undefined`。

```ts twoslash
interface Shape {}
declare function getShape(): Shape;

interface PaintOptions {
  shape: Shape;
  xPos?: number;
  yPos?: number;
}

// ---cut---
function paintShape(opts: PaintOptions) {
  let xPos = opts.xPos;
  //              ^?
  let yPos = opts.yPos;
  //              ^?
  // ...
}
```

在JavaScript中，即使我们从未设置过某一属性，我们仍然可以访问它 - 它只是会返回 `undefined` 值。
我们可以通过检查这个值来特别处理 `undefined`。

```ts twoslash
interface Shape {}
declare function getShape(): Shape;

interface PaintOptions {
  shape: Shape;
  xPos?: number;
  yPos?: number;
}

// ---cut---
function paintShape(opts: PaintOptions) {
  let xPos = opts.xPos === undefined ? 0 : opts.xPos;
  //  ^?
  let yPos = opts.yPos === undefined ? 0 : opts.yPos;
  //  ^?
  // ...
}
```

请注意，这种为未指定值设置默认值的模式非常常见，以至于 JavaScript 具有支持它的语法。

```ts twoslash
interface Shape {}
declare function getShape(): Shape;

interface PaintOptions {
  shape: Shape;
  xPos?: number;
  yPos?: number;
}

// ---cut---
function paintShape({ shape, xPos = 0, yPos = 0 }: PaintOptions) {
  console.log("x coordinate at", xPos);
  //                             ^?
  console.log("y coordinate at", yPos);
  //                             ^?
  // ...
}
```

在这里，我们使用了 [解构模式](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment) 来处理 paintShape 的参数, 并为 `xPos` 和 `yPos` 提供了 [默认值](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment#Default_values) 。
现在，在 `paintShape` 函数体内，`xPos` 和 `yPos` 都是确定存在的，但对任何调用 `paintShape` 的人来说，这两个参数是可选的。

> 请注意，目前在解构模式中没有办法使用类型注解。
> 这是因为在 JavaScript 中，以下语法已经有了不同的含义。
>
> ```ts twoslash
> // @noImplicitAny: false
> // @errors: 2552 2304
> interface Shape {}
> declare function render(x: unknown);
> // ---cut---
> function draw({ shape: Shape, xPos: number = 100 /*...*/ }) {
>   render(shape);
>   render(xPos);
> }
> ```
>
> 在对象解构模式中，`shape: Shape` 意味着“获取 `shape` 属性并在本地将其重新定义为名为 `Shape` 的变量”。
> 同样地，`xPos: number` 创建了一个名为 `number` 的变量，其值基于参数的 `xPos`。

### `readonly` 属性

属性也可以在 TypeScript 中被标记为 `readonly`。
虽然在运行时它不会改变任何行为，但在类型检查期间，标记为 `readonly` 的属性不能被修改。

```ts twoslash
// @errors: 2540
interface SomeType {
  readonly prop: string;
}

function doSomething(obj: SomeType) {
  // 我们可以读取 'obj.prop'.
  console.log(`prop has the value '${obj.prop}'.`);

  // 但是我们不能对 'obj.prop' 重新赋值
  obj.prop = "hello";
}
```

使用 `readonly` 修饰符并不意味着一个值是完全不可变的，换言之，不意味着它的内部内容不能被更改。
它只是表示属性本身不能被重新赋值。

```ts twoslash
// @errors: 2540
interface Home {
  readonly resident: { name: string; age: number };
}

function visitForBirthday(home: Home) {
  // We can read and update properties from 'home.resident'.
  console.log(`Happy birthday ${home.resident.name}!`);
  home.resident.age++;
}

function evict(home: Home) {
  // But we can't write to the 'resident' property itself on a 'Home'.
  home.resident = {
    name: "Victor the Evictor",
    age: 42,
  };
}
```

理解 `readonly` 暗示的含义是非常重要的。
它在开发期间有助于向 TypeScript 表示如何使用一个对象的意图。
在检查两种类型是否兼容时，TypeScript 不会考虑这两种类型的属性是否为 `readonly`，因此，通过别名，`readonly` 属性也可能改变。

```ts twoslash
interface Person {
  name: string;
  age: number;
}

interface ReadonlyPerson {
  readonly name: string;
  readonly age: number;
}

let writablePerson: Person = {
  name: "Person McPersonface",
  age: 42,
};

// works
let readonlyPerson: ReadonlyPerson = writablePerson;

console.log(readonlyPerson.age); // prints '42'
writablePerson.age++;
console.log(readonlyPerson.age); // prints '43'
```

通过使用 [映射修饰符](/zh/docs/handbook/2/mapped-types.html#mapping-modifiers), 你可以移除 `readonly` 属性。

### 索引签名

有时候你可能事先并不知道类型的所有属性名称，但你知道值的结构。

在这种情况下，你可以使用索引签名来描述可能值的类型，例如：

```ts twoslash
declare function getStringArray(): StringArray;
// ---cut---
interface StringArray {
  [index: number]: string;
}

const myArray: StringArray = getStringArray();
const secondItem = myArray[1];
//     ^?
```

上面，我们有一个带有索引签名的 `StringArray` 接口。
这个索引签名表明当一个 `StringArray` 用 `number` 进行索引时，它将返回一个 `string`。

只有一些类型被允许用于索引签名属性：`string`，`number`，`symbol`，模板字符串模式，以及仅由这些组成的联合类型。

<details>
    <summary>支持多种类型的索引器是可能的...</summary>
    <p>可以支持多种类型的索引器。注意，当同时使用 `number` 和 `string` 索引器时，从数字索引器返回的类型必须是从字符串索引器返回的类型的子类型。这是因为当用 <code>number</code>, 进行索引时，JavaScript 实际上会先将其转换为 <code>string</code>，然后再对对象进行索引。这意味着用 code>100</code>（一个 <code>number</code>）进行索引和用 <code>"100"</code>（一个 <code>string</code>）进行索引是相同的，所以两者需要保持一致。 </p>

```ts twoslash
// @errors: 2413
// @strictPropertyInitialization: false
interface Animal {
  name: string;
}

interface Dog extends Animal {
  breed: string;
}

// Error: indexing with a numeric string might get you a completely separate type of Animal!
interface NotOkay {
  [x: number]: Animal;
  [x: string]: Dog;
}
```

</details>

虽然字符串索引签名是描述"字典"模式的有效方式，但它们也要求所有属性与其返回类型相匹配。
这是因为字符串索引声明了 `obj.property` 也可以作为 `obj["property"]` 使用。
在以下示例中，name 的类型与字符串索引的类型不匹配，类型检查器会给出一个错误：

```ts twoslash
// @errors: 2411
// @errors: 2411
interface NumberDictionary {
  [index: string]: number;

  length: number; // ok
  name: string;
}
```

然而，如果索引签名是属性类型的联合，那么接受不同类型的属性是可以的。

```ts twoslash
interface NumberOrStringDictionary {
  [index: string]: number | string;
  length: number; // ok, length is a number
  name: string; // ok, name is a string
}
```

最后，你可以将索引签名设为 `readonly`，以阻止对其索引进行赋值。

```ts twoslash
declare function getReadOnlyStringArray(): ReadonlyStringArray;
// ---cut---
// @errors: 2542
interface ReadonlyStringArray {
  readonly [index: number]: string;
}

let myArray: ReadonlyStringArray = getReadOnlyStringArray();
myArray[2] = "Mallory";
```

你不能设置 `myArray[2]`，因为索引签名是 `readonly`。

## 严格属性检查

对象被赋予类型的地方和方式在类型系统中可能会产生差异。
其中的一个关键例子就是严格属性检查，在对象创建并在创建过程中被赋予对象类型时，它会对对象进行更彻底的验证。

```ts twoslash
// @errors: 2345 2739
interface SquareConfig {
  color?: string;
  width?: number;
}

function createSquare(config: SquareConfig): { color: string; area: number } {
  return {
    color: config.color || "red",
    area: config.width ? config.width * config.width : 20,
  };
}

let mySquare = createSquare({ colour: "red", width: 100 });
```

注意给 `createSquare` 的参数拼写成了 `colour` 而不是 `color`。 
在纯 JavaScript 中，这种情况会静默失败。

你可能会认为这个程序的类型是正确的，因为 `width` 属性是兼容的，没有 `color` 属性存在，而额外的 `colour` 属性是不重要的。

然而，TypeScript 的立场是这段代码可能有个错误。
对象字面量在赋值给其他变量或作为参数传递时，会得到特殊处理并进行 _严格属性检查_ 。
如果一个对象字面量有任何 "目标类型" 所没有的属性，你就会得到一个错误：

```ts twoslash
// @errors: 2345 2739
interface SquareConfig {
  color?: string;
  width?: number;
}

function createSquare(config: SquareConfig): { color: string; area: number } {
  return {
    color: config.color || "red",
    area: config.width ? config.width * config.width : 20,
  };
}
// ---cut---
let mySquare = createSquare({ colour: "red", width: 100 });
```

实际上，绕过这些检查非常简单。
最简单的方法就是使用类型断言：

```ts twoslash
// @errors: 2345 2739
interface SquareConfig {
  color?: string;
  width?: number;
}

function createSquare(config: SquareConfig): { color: string; area: number } {
  return {
    color: config.color || "red",
    area: config.width ? config.width * config.width : 20,
  };
}
// ---cut---
let mySquare = createSquare({ width: 100, opacity: 0.5 } as SquareConfig);
```

然而，如果你确定对象可以有一些在某些特殊方式上使用的额外属性，那么更好的方法可能是添加一个字符串索引签名。
如果 `SquareConfig` 可以有上述类型的 `color` 和 `width` 属性，但 _也_ 可能有其他任意数量的属性，那么我们可以这么定义它：

```ts twoslash
interface SquareConfig {
  color?: string;
  width?: number;
  [propName: string]: any;
}
```

这里我们指定了 `SquareConfig` 可以拥有任意数量的属性，只要他们不是 `color` 或 `width`，那么这些属性的类型就无关紧要。

最后一个可以绕过这种检查的方法，可能有些出乎意料，就是把对象赋值给另一个变量：
因为在赋值 `squareOptions` 的过程中不会进行严格属性检查，所以编译器不会报出错误：

```ts twoslash
interface SquareConfig {
  color?: string;
  width?: number;
  [propName: string]: any;
}

function createSquare(config: SquareConfig): { color: string; area: number } {
  return {
    color: config.color || "red",
    area: config.width ? config.width * config.width : 20,
  };
}
// ---cut---
let squareOptions = { colour: "red", width: 100 };
let mySquare = createSquare(squareOptions);
```

上述解决方案只有在 `squareOptions` 和 `SquareConfig` 之间存在共享属性的时候能够生效。
在这个例子中，共享的属性是 `width`。然而，如果变量没有任何的共享对象属性，这种方法就会失败。例如：

```ts twoslash
// @errors: 2559
interface SquareConfig {
  color?: string;
  width?: number;
}

function createSquare(config: SquareConfig): { color: string; area: number } {
  return {
    color: config.color || "red",
    area: config.width ? config.width * config.width : 20,
  };
}
// ---cut---
let squareOptions = { colour: "red" };
let mySquare = createSquare(squareOptions);
```

对于简单的代码，你通常不需要尝试“绕过”这些检查。
对于包含方法和持有状态的更复杂的对象字面量，你可能需要记住这些技巧，但大多数严格属性错误实际上是bug。

这意味着，如果你在处理类似选项包的严格属性检查问题时，可能需要修改一些类型声明。
在这种情况下，如果允许向 `createSquare` 传递一个同时具有 `color` 或 `colour` 属性的对象，你应该修正 `SquareConfig` 的定义以反映这一点。

## 扩展类型

对于某些类型有更特殊版本的需求是很常见的事情。
例如，我们可能有一个叫做 `BasicAddress` 的类型，用来描述在美国发送信件和包裹所需要的字段。

```ts twoslash
interface BasicAddress {
  name?: string;
  street: string;
  city: string;
  country: string;
  postalCode: string;
}
```

在某些情况下，这已经足够了，但如果一个地址的建筑有多个单元，通常还会有一个单元号与之关联。
我们可以进一步描述一个 `AddressWithUnit`。

<!-- prettier-ignore -->
```ts twoslash
interface AddressWithUnit {
  name?: string;
  unit: string;
//^^^^^^^^^^^^^
  street: string;
  city: string;
  country: string;
  postalCode: string;
}
```

这段代码的功能是能完成任务，但缺点是我们不得不重复 `BasicAddress` 中的所有其他字段，尽管我们的修改完全是附加的。
相反，我们可以扩展原始的 `BasicAddress` 类型，并仅添加对 `AddressWithUnit` 独有的新字段。

```ts twoslash
interface BasicAddress {
  name?: string;
  street: string;
  city: string;
  country: string;
  postalCode: string;
}

interface AddressWithUnit extends BasicAddress {
  unit: string;
}
```

`extends` 关键词在 `interface` 中的使用可以让我们有效地从其他已命名类型中复制其成员，并添加任何新的成员。
这对减少我们写的类型声明样板代码量，以及传递出多个不同的具有相同属性的声明可能相关的目标，非常有用。
例如，`AddressWithUnit` 没有需要重复 `street` 属性，而且因为 `street` 源自 `BasicAddress`，读者会知道这两种类型在某种程度上是相互关联的。

另外，`interface` 还可以扩展自多个类型。

```ts twoslash
interface Colorful {
  color: string;
}

interface Circle {
  radius: number;
}

interface ColorfulCircle extends Colorful, Circle {}

const cc: ColorfulCircle = {
  color: "red",
  radius: 42,
};
```

## 交叉(Intersections)类型

`interface` 允许我们通过扩展现有类型来构建新的类型。
TypeScript 提供了另一种被称为 _交叉类型_ 的结构，主要用于合并现有的对象类型。

交叉类型使用 `&` 运算符定义。

```ts twoslash
interface Colorful {
  color: string;
}
interface Circle {
  radius: number;
}

type ColorfulCircle = Colorful & Circle;
```

在这里，我们将 `Colorful` 和 `Circle` 进行了交叉，从而产生了一个新的类型，这个新类型具有 `Colorful` _和_ `Circle` 所有的成员属性。

```ts twoslash
// @errors: 2345
interface Colorful {
  color: string;
}
interface Circle {
  radius: number;
}
// ---cut---
function draw(circle: Colorful & Circle) {
  console.log(`Color was ${circle.color}`);
  console.log(`Radius was ${circle.radius}`);
}

// okay
draw({ color: "blue", radius: 42 });

// oops
draw({ color: "red", raidus: 42 });
```

## 接口 vs. 交叉类型 (Interfaces vs. Intersections)

我们刚刚看到了两种组合类型的方法，这两种方法看起来很相似，但实际上有微妙的差别。
在接口中，我们可以使用 `extends` 子句从其他类型扩展，我们也可以使用交叉类型做类似的操作，并使用类型别名命名结果。
这两者的主要区别在于如何处理冲突，这通常是你在接口和交叉类型的类型别名之间进行选择的主要原因。

如果定义了具有相同名称的接口，TypeScript 会尝试合并它们（只要它们的属性是兼容的）。如果属性不兼容（即，它们具有相同的属性名但类型不同），TypeScript 将会抛出错误。

在交叉类型的情况下，不同类型的属性会被自动合并。当后面使用这种类型时，TypeScript 会期望该属性同时满足两种类型，这可能产生意外的结果。

例如，以下代码会抛出错误，因为这些属性是不兼容的：

```ts
interface Person {
  name: string;
}

interface Person {
  name: number;
}
```

相反，下面的代码可以编译，但它会返回一个 never 类型：

```ts twoslash
interface Person1 {
  name: string;
}

interface Person2 {
  name: number;
}

type Staff = Person1 & Person2

declare const staffer: Staff;
staffer.name;
//       ^?
```
在这种情况下，Staff 需要 name 属性同时为字符串和数字类型，这会导致属性的类型为 `never`。

## 泛型对象类型

设想一个 `Box` 类型，它可以包含任何值 —— `string`、`number`、`Giraffe` 等，没有限制。

```ts twoslash
interface Box {
  contents: any;
}
```

现在，`contents` 属性的类型被定义为 `any`，这是可以工作的，但在使用过程中可能导致一些意外。

我们可以选择使用 `unknown` 替代 `any`，但这就意味着在我们已经知道 `contents` 类型的情况下，我们需要进行预防性检查，或使用容易出错的类型断言。

```ts twoslash
interface Box {
  contents: unknown;
}

let x: Box = {
  contents: "hello world",
};

// we could check 'x.contents'
if (typeof x.contents === "string") {
  console.log(x.contents.toLowerCase());
}

// or we could use a type assertion
console.log((x.contents as string).toLowerCase());
```

一个类型安全的方法是为每一种 `contents` 都搭建出不同的 `Box` 类型。

```ts twoslash
// @errors: 2322
interface NumberBox {
  contents: number;
}

interface StringBox {
  contents: string;
}

interface BooleanBox {
  contents: boolean;
}
```

但这意味着我们必须创建不同的函数，或者为这些类型创建函数的重载来进行操作。

```ts twoslash
interface NumberBox {
  contents: number;
}

interface StringBox {
  contents: string;
}

interface BooleanBox {
  contents: boolean;
}
// ---cut---
function setContents(box: StringBox, newContents: string): void;
function setContents(box: NumberBox, newContents: number): void;
function setContents(box: BooleanBox, newContents: boolean): void;
function setContents(box: { contents: any }, newContents: any) {
  box.contents = newContents;
}
```

这里有大量的样板代码。此外，我们可能稍后需要引入新的类型和重载。
这很令人沮丧，因为我们的 box 类型和重载实际上都是相同的。

相反，我们可以创建一个声明 _类型参数_ 的通用 `box` 类型。

```ts twoslash
interface Box<Type> {
  contents: Type;
}
```

你可以将这段解读为：“一个 `Box<Type>` 是一个 `contents` 为 `Type` 类型的对象。”
稍后，在我们引用到 `Box` 的地方，我们需要在 `Type` 所在位置提供一个 _类型参数_ `Type`。

```ts twoslash
interface Box<Type> {
  contents: Type;
}
// ---cut---
let box: Box<string>;
```

你可以将 `Box` 视为一种真实类型的模板，其中的 `Type` 为某种即将被替换为其它类型的占位符。
当 TypeScript 看到 `Box<string>` 的时候，它将会把 `Box<Type>` 中的每一个 `Type` 替换为`string`，并且最后得到一个类似 `{ contents: string }` 的结果。
换句话说，`Box<string>` 和我们之前的 `StringBox` 的工作方式是相同的。

```ts twoslash
interface Box<Type> {
  contents: Type;
}
interface StringBox {
  contents: string;
}

let boxA: Box<string> = { contents: "hello" };
boxA.contents;
//   ^?

let boxB: StringBox = { contents: "world" };
boxB.contents;
//   ^?
```

`Box` 类型的可重用性在于可以用任何东西替换 `Type`。这意味着当我们需要一个新类型的 `Box` 时，我们根本不需要声明一个新的 `Box` 类型（尽管如果我们想要的话，当然也可以这样做）。

```ts twoslash
interface Box<Type> {
  contents: Type;
}

interface Apple {
  // ....
}

// Same as '{ contents: Apple }'.
type AppleBox = Box<Apple>;
```

这也意味着我们可以完全避免使用重载，转而使用[泛型函数](/docs/handbook/2/functions.html#generic-functions)。

```ts twoslash
interface Box<Type> {
  contents: Type;
}

// ---cut---
function setContents<Type>(box: Box<Type>, newContents: Type) {
  box.contents = newContents;
}
```

值得注意的是，类型别名也可以是泛型。我们本可以使用类型别名来定义我们的新 `Box<Type>` 接口，它是这样的：

```ts twoslash
interface Box<Type> {
  contents: Type;
}
```

但改为使用类型别名定义也是可以的：

```ts twoslash
type Box<Type> = {
  contents: Type;
};
```

由于类型别名不仅仅可以描述对象类型，我们也可以用它们编写其他种类的泛型辅助类型

```ts twoslash
// @errors: 2575
type OrNull<Type> = Type | null;

type OneOrMany<Type> = Type | Type[];

type OneOrManyOrNull<Type> = OrNull<OneOrMany<Type>>;
//   ^?

type OneOrManyOrNullStrings = OneOrManyOrNull<string>;
//   ^?
```

我们稍后将回到类型别名的话题。

### `Array` 类型

泛型对象类型通常作为某种容器类型，它们的工作方式独立于它们所包含的元素的类型。
最理想的是让数据结构以这种方式工作，这样它们可以在不同的数据类型之间重用。

我们在整个手册中一直在使用这样的类型：`Array` 类型。
每当我们写出像 `number[]` 或 `string[]` 这样的类型时，这实际上就是 `Array<number>` 和 `Array<string>` 的简写。

```ts twoslash
function doSomething(value: Array<string>) {
  // ...
}

let myArray: string[] = ["hello", "world"];

// either of these work!
doSomething(myArray);
doSomething(new Array("hello", "world"));
```

就像上面的 `Box` 类型一样，`Array` 本身也是一个泛型类型。

```ts twoslash
// @noLib: true
interface Number {}
interface String {}
interface Boolean {}
interface Symbol {}
// ---cut---
interface Array<Type> {
  /**
   * Gets or sets the length of the array.
   */
  length: number;

  /**
   * Removes the last element from an array and returns it.
   */
  pop(): Type | undefined;

  /**
   * Appends new elements to an array, and returns the new length of the array.
   */
  push(...items: Type[]): number;

  // ...
}
```

现代 JavaScript 还提供了其他的泛型数据结构，比如 `Map<K, V>`，`Set<T>`，以及 `Promise<T>`。
这实际上意味着，由于 `Map`，`Set` 和 `Promise` 的行为方式，它们可以与任何类型的数据集合一起工作。

### `ReadonlyArray` 类型

`ReadonlyArray` 是一个特殊的类型，用来描述那些不应该被更改的数组。

```ts twoslash
// @errors: 2339
function doStuff(values: ReadonlyArray<string>) {
  // We can read from 'values'...
  const copy = values.slice();
  console.log(`The first value is ${values[0]}`);

  // ...but we can't mutate 'values'.
  values.push("hello!");
}
```

正如属性的 `readonly` 修饰符一样，它主要是我们用来表达意图的工具。
当我们看到一个函数返回 `ReadonlyArray` 时，这告诉我们我们不应该更改数组的内容。而当我们看到一个函数使用 `ReadonlyArray` 时，这告诉我们我们可以将任何数组传递给该函数，而不用担心它会改变数组的内容。

与 `Array` 不同的是，`ReadonlyArray` 没有我们可以使用的构造函数。

```ts twoslash
// @errors: 2693
new ReadonlyArray("red", "green", "blue");
```

Instead, we can assign regular `Array`s to `ReadonlyArray`s.

```ts twoslash
const roArray: ReadonlyArray<string> = ["red", "green", "blue"];
```

正如 TypeScript 为 `Array<Type>` 提供了简写语法 `Type[]`，它也为 `ReadonlyArray<Type>` 提供了简写语法 `readonly Type[]`。

```ts twoslash
// @errors: 2339
function doStuff(values: readonly string[]) {
  //                     ^^^^^^^^^^^^^^^^^
  // We can read from 'values'...
  const copy = values.slice();
  console.log(`The first value is ${values[0]}`);

  // ...but we can't mutate 'values'.
  values.push("hello!");
}
```

需要再强调一点的是，不像 `readonly` 属性修饰符，常规数组与 `ReadonlyArray` 之间的可分配性并不是双向的。

```ts twoslash
// @errors: 4104
let x: readonly string[] = [];
let y: string[] = [];

x = y;
y = x;
```

### 元组类型(Tuple Types)

_元组类型_ 是另一种 `Array` 类型，它知道自身包含的元素数量，以及各个位置上精确的数据类型。

```ts twoslash
type StringNumberPair = [string, number];
//                      ^^^^^^^^^^^^^^^^
```

在这里，`StringNumberPair` 是由 `string` 和 `number` 构成的元组类型。
就像 `ReadonlyArray` 一样，它在运行时没有任何表现，但对 TypeScript 来说它是很重要的。
对类型系统来说，`StringNumberPair` 描述的是数组，其第 `0` 个索引处包含一个 `string` 类型的值，而第 `1` 个索引处包含一个 `number` 类型的值。

```ts twoslash
function doSomething(pair: [string, number]) {
  const a = pair[0];
  //    ^?
  const b = pair[1];
  //    ^?
  // ...
}

doSomething(["hello", 42]);
```

如果我们尝试超出元素数量的索引，我们将会得到一个错误:

```ts twoslash
// @errors: 2493
function doSomething(pair: [string, number]) {
  // ...

  const c = pair[2];
}
```

我们也可以使用 JavaScript 的数组解构对[元组进行解构](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment#Array_destructuring)。

```ts twoslash
function doSomething(stringHash: [string, number]) {
  const [inputString, hash] = stringHash;

  console.log(inputString);
  //          ^?

  console.log(hash);
  //          ^?
}
```

> 元组类型在基于约定的API中非常有用，其中每个元素的含义都是“显而易见”的。
> 这使我们在解构它们时可以灵活地命名变量。
> 在上述示例中，我们可以自由地为元素0和1命名。
>
> 然而，由于不是每个用户都对什么是显而易见有相同的看法，因此可能值得重新考虑是否使用具有描述性属性名称的对象可能更适合您的 API。

除了长度检查之外，像这样的简单元组类型等同于声明了特定索引属性的 `Array` 类型版本，并且声明了具有数值字面量类型的 `length`。

```ts twoslash
interface StringNumberPair {
  // specialized properties
  length: 2;
  0: string;
  1: number;

  // Other 'Array<string | number>' members...
  slice(start?: number, end?: number): Array<string | number>;
}
```

你可能对此也感兴趣：元组可以通过在元素类型后添加问号(`?`)的方式设定可选属性。
可选的元组元素只能出现在末尾，并且也会影响 `length` 的类型。

```ts twoslash
type Either2dOr3d = [number, number, number?];

function setCoordinate(coord: Either2dOr3d) {
  const [x, y, z] = coord;
  //           ^?

  console.log(`Provided coordinates had ${coord.length} dimensions`);
  //                                            ^?
}
```

元组也可以有剩余元素（reset elements），这些元素必须是一个数组/元组类型。

```ts twoslash
type StringNumberBooleans = [string, number, ...boolean[]];
type StringBooleansNumber = [string, ...boolean[], number];
type BooleansStringNumber = [...boolean[], string, number];
```

- `StringNumberBooleans` 描述的是一个元组，其第一和第二元素分别是 `string` 和 `number`，但是其后可能跟着任意数量的 `boolean`。
- `StringBooleansNumber` 描述的是一个元组，其第一个元素是 `string`，然后是任意数量的 `boolean`，最后是一个 `number`。
- `BooleansStringNumber` 描述的是一个元组，其开始元素是任意数量的 `boolean`，然后是一个 `string`，最后是一个 `number`。

一个带有剩余元素的元组没有固定的"长度"——它只有在不同位置的一组已知元素。

```ts twoslash
type StringNumberBooleans = [string, number, ...boolean[]];
// ---cut---
const a: StringNumberBooleans = ["hello", 1];
const b: StringNumberBooleans = ["beautiful", 2, true];
const c: StringNumberBooleans = ["world", 3, true, false, true, false, true];
```

为什么可选元素和剩余元素可能有用？
答：这使得 TypeScript 可以把元组与参数列表相对应。
元组类型可以被用于[剩余参数和参数传值](/zh/docs/handbook/2/functions.html#rest-parameters-and-arguments)，于是以下内容：

```ts twoslash
function readButtonInput(...args: [string, number, ...boolean[]]) {
  const [name, version, ...input] = args;
  // ...
}
```

基本上等同于：

```ts twoslash
function readButtonInput(name: string, version: number, ...input: boolean[]) {
  // ...
}
```

这在你想使用 rest 参数来接收可变数量的参数，并且你需要一定数量的元素，但你不希望引入中间变量时非常方便。

<!--
TODO do we need this example?

For example, imagine we need to write a function that adds up `number`s based on arguments that get passed in.

```ts twoslash
function sum(...args: number[]) {
    // ...
}
```

We might feel like it makes little sense to take any fewer than 2 elements, so we want to require callers to provide at least 2 arguments.
A first attempt might be

```ts twoslash
function foo(a: number, b: number, ...args: number[]) {
    args.unshift(a, b);

    let result = 0;
    for (const value of args) {
        result += value;
    }
    return result;
}
```

-->

### `readonly` 元组类型

关于元组类型的最后一点说明 - 元组类型有 `readonly `变体，可以通过在它们前面添加 `readonly` 修饰符来指定，就像数组简写语法一样。

```ts twoslash
function doSomething(pair: readonly [string, number]) {
  //                       ^^^^^^^^^^^^^^^^^^^^^^^^^
  // ...
}
```

正如你可能预期的，TypeScript 不允许写入 `readonly` 元组的任何属性。

```ts twoslash
// @errors: 2540
function doSomething(pair: readonly [string, number]) {
  pair[0] = "hello!";
}
```

在大多数代码中，元组往往被创建出来并且不会被修改，因此，当可能的时候，将类型作为 `readonly` 元组进行标注是一个好的默认选择。
这也非常重要，因为带有 `const` 断言的数组字面量将被推断为 `readonly `元组类型。

```ts twoslash
// @errors: 2345
let point = [3, 4] as const;

function distanceFromOrigin([x, y]: [number, number]) {
  return Math.sqrt(x ** 2 + y ** 2);
}

distanceFromOrigin(point);
```

这里，`distanceFromOrigin` 从不修改其元素，但却期望一个可变的元组。
由于 `point` 的类型被推断为 `readonly [3, 4]`，它与 `[number, number]` 类型不兼容，因为后者不能保证 `point` 的元素不会被修改。

<!-- ## Other Kinds of Object Members

Most of the declarations in object types:

### Method Syntax

### Call Signatures

### Construct Signatures

### Index Signatures -->
