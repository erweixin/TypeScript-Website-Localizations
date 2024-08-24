---
title: 范型
layout: docs
permalink: /zh/docs/handbook/2/generics.html
oneline: 可接收参数的类型
---

软件工程的一个重要部分是构建出具有定义明确且一致的 API，而且可复用的组件。
能够处理现今和未来的数据的组件会为你构建大规模软件系统提供最灵活的能力。

在 C# 和 Java 这样的语言中，创建可复用组件的主要工具之一就是 _泛型_。也就是说，你可以创建一个能够处理各种类型，而不仅仅是一种类型的组件。
这样，用户就可以使用这些组件并应用他们自己的类型。

## 泛型的 "Hello World"

首先，我们来做泛型的 "Hello World"：身份函数（the identity function）。
身份函数是一个将返回任何传入参数的函数。你可以将其理解为类似于 `echo` 命令的东西。

没有泛型的话，我们必须为身份函数指定具体的类型：

```ts twoslash
function identity(arg: number): number {
  return arg;
}
```

或者，我们也可以用 `any` 类型来描述这个身份函数：

```ts twoslash
function identity(arg: any): any {
  return arg;
}
```

尽管使用 `any` 的确是一种泛型，因为它会让函数接受任何类型作为 `arg` 的类型，但实际上，当函数返回时，我们失去了关于输入类型的信息。
如果我们传入一个数字，我们只知道返回的可能是任何类型。

相反，我们需要一种捕获参数类型的方式，以便我们也可以用它来表示返回什么。
这里，我们将会使用一种 _类型变量_，这是一种特殊的变量，它作用在类型上，而不是值上。

```ts twoslash
function identity<Type>(arg: Type): Type {
  return arg;
}
```

我们现在已经为身份函数添加了一个类型变量 `Type`。
这个 `Type` 允许我们捕获用户提供的类型（比如 `number`），以便我们稍后使用这些信息。
在这里，我们将 `Type` 再次用作返回类型。通过检查，我们可以看到参数类型和返回类型是相同的。
这允许我们将类型信息在函数的两个方向上进行传递。

我们称这个版本的 `identity` 函数为泛型，因为它可以在一系列类型上工作。
与使用 `any` 不同，它的准确性也一样高（即，它不会丢失任何信息），就像第一个使用数字作为参数和返回类型的 `identity` 函数那样。

一旦我们写下了泛型的身份函数，我们可以用两种方式之一来调用它。
第一种方式是将所有的参数，包括类型参数，都传递给函数：

```ts twoslash
function identity<Type>(arg: Type): Type {
  return arg;
}
// ---cut---
let output = identity<string>("myString");
//       ^?
```

在这里，我们明确将 `Type` 设置为 `string`，作为函数调用的参数之一，以 `<>` 标记参数，而不是用 `()`。

第二种方式也许是最常见的。在这里，我们使用 _类型参数推断_ -- 也就是说，我们希望编译器能根据我们传入的参数的类型自动为我们设置 `Type` 的值：

```ts twoslash
function identity<Type>(arg: Type): Type {
  return arg;
}
// ---cut---
let output = identity("myString");
//       ^?
```

注意，我们没有必要在尖括号（`<>`）中明确地传递类型；编译器只是查看了值 `"myString"`，并将 `Type` 设为了它的类型。
虽然类型参数推断可以作为有用的工具，保持代码的短小和可读性，但是当编译器无法推断出类型时，你可能需要像我们在前一个例子中那样显式地传入类型参数，比如在更复杂的示例中可能会发生这种情况。

## 使用泛型类型变量

当你开始使用泛型时，你会注意到，当你创建像 identity 这样的泛型函数时，编译器会强制你在函数体内正确地使用任何泛型类型的参数。
也就是说，你需要将这些参数视为可能是任何类型。

我们来看看之前的 `identity` 函数：

```ts twoslash
function identity<Type>(arg: Type): Type {
  return arg;
}
```

如果我们同时还想在每次调用过程中，将参数 `arg` 的长度输出到控制台，
我们可能会尝试这样写：

```ts twoslash
// @errors: 2339
function loggingIdentity<Type>(arg: Type): Type {
  console.log(arg.length);
  return arg;
}
```

当我们这样做的时候，编译器会给我们一个错误，我们在使用 `arg` 的 `.length` 属性，但我们并没有说 `arg` 有这个属性。记住，我们早就说过这些类型变量代表了任何类型，所以使用这个函数的人可能已经传入一个 `number` 作为参数，而 `number` 是没有 `.length` 属性的。

假设我们实际上是想让这个函数处理 `Type` 类型的数组，而不是直接处理 `Type`。既然我们处理的是数组，`.length` 属性应该是可用的。我们可以像创建其他类型的数组一样来描述这种情况：

```ts twoslash {1}
function loggingIdentity<Type>(arg: Type[]): Type[] {
  console.log(arg.length);
  return arg;
}
```

你可以这样理解 `loggingIdentity` 的类型："泛型函数 `loggingIdentity` 接受一个类型参数 `Type`，以及一个 `Type` 类型元素组成的数组参数 `arg`，并返回一个 `Type` 类型元素组成的数组。"

如果我们传入一个数字数组，我们就会得到一个数字数组作为返回值，因为此时 `Type` 会被绑定为 `number` 类型。

这种方式让我们能够将泛型类型变量 `Type` 作为我们正在处理的类型的一部分，而不是整个类型，从而给我们带来更大的灵活性。

我们还可以用另一种方式来写这个示例：

```ts twoslash {1}
function loggingIdentity<Type>(arg: Array<Type>): Array<Type> {
  console.log(arg.length); // Array has a .length, so no more error
  return arg;
}
```

你可能已经在其他编程语言中见过这种类型风格。
在下一节中，我们将讨论如何创建像 `Array<Type>` 这样的自定义泛型类型。

## 泛型类型

在前面的章节中，我们创建了可以处理多种类型的泛型身份函数。
在本节中，我们将深入探讨函数本身的类型，以及如何创建泛型接口。

泛型函数的类型声明方式与非泛型函数类似，只是需要先列出类型参数，这一点与函数声明的语法相似：

```ts twoslash
function identity<Type>(arg: Type): Type {
  return arg;
}

let myIdentity: <Type>(arg: Type) => Type = identity;
```

我们其实也可以在类型中使用不同的名称来表示泛型类型参数，只要类型变量的数量和使用方式保持一致即可。

```ts twoslash
function identity<Type>(arg: Type): Type {
  return arg;
}

let myIdentity: <Input>(arg: Input) => Input = identity;
```

我们还可以将泛型类型写成对象字面量类型的调用签名：

```ts twoslash
function identity<Type>(arg: Type): Type {
  return arg;
}

let myIdentity: { <Type>(arg: Type): Type } = identity;
```

这就引导我们写出了第一个泛型接口。
让我们把前面例子中的对象字面量转换成一个 interface：

```ts twoslash
interface GenericIdentityFn {
  <Type>(arg: Type): Type;
}

function identity<Type>(arg: Type): Type {
  return arg;
}

let myIdentity: GenericIdentityFn = identity;
```

在一个类似的例子中，我们可能想把泛型参数变成整个接口的参数。
这样做可以让我们清楚地看到接口是基于哪种类型（或哪些类型）的泛型（比如 `Dictionary<string>` 而不仅仅是 `Dictionary`）。
这种方式使得类型参数对接口的所有其他成员都可见。

```ts twoslash
interface GenericIdentityFn<Type> {
  (arg: Type): Type;
}

function identity<Type>(arg: Type): Type {
  return arg;
}

let myIdentity: GenericIdentityFn<number> = identity;
```

请注意，我们的例子现在略有变化。
我们不再描述一个泛型函数，而是有了一个作为泛型类型一部分的非泛型函数签名。
当我们使用 `GenericIdentityFn` 时，现在还需要指定相应的类型参数（在这里是 `number`），这实际上锁定了底层调用签名将使用的类型。
了解何时将类型参数直接放在调用签名上，何时将其放在接口本身上，将有助于描述类型的哪些方面是泛型的。

除了泛型接口，我们还可以创建泛型类。
需要注意的是，无法创建泛型枚举和泛型命名空间。

## 泛型类

泛型类的结构与泛型接口类似。
泛型类在类名后面有一个用尖括号（`<>`）括起来的泛型类型参数列表。

```ts twoslash
// @strict: false
class GenericNumber<NumType> {
  zeroValue: NumType;
  add: (x: NumType, y: NumType) => NumType;
}

let myGenericNumber = new GenericNumber<number>();
myGenericNumber.zeroValue = 0;
myGenericNumber.add = function (x, y) {
  return x + y;
};
```

这是对 `GenericNumber` 类的一种相当直接的使用，但你可能已经注意到，没有任何限制要求它只能使用 `number` 类型。
我们其实可以用 `string` 类型，甚至可以使用更复杂的对象类型。

```ts twoslash
// @strict: false
class GenericNumber<NumType> {
  zeroValue: NumType;
  add: (x: NumType, y: NumType) => NumType;
}
// ---cut---
let stringNumeric = new GenericNumber<string>();
stringNumeric.zeroValue = "";
stringNumeric.add = function (x, y) {
  return x + y;
};

console.log(stringNumeric.add(stringNumeric.zeroValue, "test"));
```

就像接口一样，将类型参数放在类本身上可以确保类的所有属性都使用相同的类型。

正如我们在[类的章节中](/zh/docs/handbook/2/classes.html)所介绍的，一个类的类型有两个方面：静态部分和实例部分。
泛型类只在其实例部分是泛型的，而不是在其静态部分。因此，在使用类时，静态成员不能使用类的类型参数。

## 泛型约束

还记得之前的例子吗？有时候你可能想要编写一个泛型函数，这个函数能够处理一组类型，而你对这组类型的某些特性有一定了解。

拿我们的 `loggingIdentity` 函数来说，我们想要访问 `arg` 的 `.length` 属性。但是编译器无法确定每种类型都有 `.length` 属性，所以它会警告我们不能做这种假设。

```ts twoslash
// @errors: 2339
function loggingIdentity<Type>(arg: Type): Type {
  console.log(arg.length);
  return arg;
}
```

我们不希望这个函数可以处理任何类型，而是想要限制它只能处理那些同时具有 `.length` 属性的类型。

只要一个类型有这个成员，我们就允许使用它，但这个成员是必需的。

要实现这一点，我们需要将这个要求列为 `Type` 可以是什么的约束。

具体做法是，我们先创建一个接口来描述这个约束。
在这里，我们会创建一个只有单个 `.length` 属性的接口，然后使用这个接口和 `extends` 关键字来表示我们的约束：

```ts twoslash
interface Lengthwise {
  length: number;
}

function loggingIdentity<Type extends Lengthwise>(arg: Type): Type {
  console.log(arg.length); // 现在我们知道它有一个 .length 属性，所以不再有错误
  return arg;
}
```

由于这个泛型函数现在受到了约束，它就不再能够适用于任何类型了：

```ts twoslash
// @errors: 2345
interface Lengthwise {
  length: number;
}

function loggingIdentity<Type extends Lengthwise>(arg: Type): Type {
  console.log(arg.length);
  return arg;
}
// ---cut---
loggingIdentity(3);
```

相反，我们需要传入的值的类型必须具备所有所需的属性：

```ts twoslash
interface Lengthwise {
  length: number;
}

function loggingIdentity<Type extends Lengthwise>(arg: Type): Type {
  console.log(arg.length);
  return arg;
}
// ---cut---
loggingIdentity({ length: 10, value: 3 });
```

## 在泛型约束中使用类型参数

我们可以声明一个类型参数，让它受到另一个类型参数的约束。
举个例子，假设我们想根据属性名从一个对象中获取属性值。
我们希望确保不会意外地获取到一个在 `obj` 对象上不存在的属性，因此我们可以在两个类型之间设置一个约束：

```ts twoslash
// @errors: 2345
function getProperty<Type, Key extends keyof Type>(obj: Type, key: Key) {
  return obj[key];
}

let x = { a: 1, b: 2, c: 3, d: 4 };

getProperty(x, "a");
getProperty(x, "m");
```

## 在泛型中使用类类型

在 TypeScript 中使用泛型创建工厂函数时，需要通过构造函数来引用类类型。例如：

```ts twoslash
function create<Type>(c: { new (): Type }): Type {
  return new c();
}
```

一个更高级的例子是使用原型属性来推断和约束构造函数与类类型实例侧之间的关系。

```ts twoslash
// @strict: false
class BeeKeeper {
  hasMask: boolean = true;
}

class ZooKeeper {
  nametag: string = "Mikle";
}

class Animal {
  numLegs: number = 4;
}

class Bee extends Animal {
  numLegs = 6;
  keeper: BeeKeeper = new BeeKeeper();
}

class Lion extends Animal {
  keeper: ZooKeeper = new ZooKeeper();
}

function createInstance<A extends Animal>(c: new () => A): A {
  return new c();
}

createInstance(Lion).keeper.nametag;
createInstance(Bee).keeper.hasMask;
```

这种模式被用来实现[混合（mixins）](/zh/docs/handbook/mixins.html)设计模式。

## 泛型参数默认值

通过为泛型类型参数声明默认值，你可以使相应的类型参数变为可选。举个例子，假设有一个创建新 `HTMLElement` 的函数。不带参数调用该函数会生成一个 `HTMLDivElement`；以某个元素作为第一个参数调用函数则会生成一个该参数类型的元素。你还可以选择传入一个子元素列表。在之前，你可能需要这样定义函数：


```ts twoslash
type Container<T, U> = {
  element: T;
  children: U;
};

// ---cut---
declare function create(): Container<HTMLDivElement, HTMLDivElement[]>;
declare function create<T extends HTMLElement>(element: T): Container<T, T[]>;
declare function create<T extends HTMLElement, U extends HTMLElement>(
  element: T,
  children: U[]
): Container<T, U[]>;
```

通过使用泛型参数默认值，我们可以将其简化为：

```ts twoslash
type Container<T, U> = {
  element: T;
  children: U;
};

// ---cut---
declare function create<T extends HTMLElement = HTMLDivElement, U extends HTMLElement[] = T[]>(
  element?: T,
  children?: U
): Container<T, U>;

const div = create();
//    ^?

const p = create(new HTMLParagraphElement());
//    ^?
```

泛型参数默认值遵循以下规则：

- 如果一个类型参数有默认值，它就被视为可选的。
- 必选的类型参数不能放在可选类型参数之后。
- 类型参数的默认值必须满足该类型参数的约束（如果存在约束的话）。
- 在指定类型参数时，你只需要为必选的类型参数提供具体类型。未指定的类型参数会使用它们的默认类型。
- 如果指定了默认类型，但类型推断无法选择合适的候选类型，那么就会使用这个默认类型。
- 当一个类或接口声明与现有的类或接口声明合并时，可以为已存在的类型参数引入默认值。
- 当一个类或接口声明与现有的类或接口声明合并时，只要指定了默认值，就可以引入新的类型参数。

