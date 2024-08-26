---
title: 类
layout: docs
permalink: /zh/docs/handbook/2/classes.html
oneline: “TypeScript 中的类是如何工作”
---

<blockquote class='bg-reading'>
  <p>背景阅读:<br /><a href='https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes'>Classes (MDN)</a></p>
</blockquote>

TypeScript 完全支持 ES2015 中引入的“class”关键字。

与其他 JavaScript 语言功能一样，TypeScript 添加了类型注释和其他语法，以允许您表达类和其他类型之间的关系。

## 类成员

这是最基本的类 - 一个空类：

```ts twoslash
class Point {}
```

这个类现在还没有什么用，让我们开始添加一些成员。

### 字段

字段声明在类上创建一个公共可写属性：

```ts twoslash
// @strictPropertyInitialization: false
class Point {
  x: number;
  y: number;
}

const pt = new Point();
pt.x = 0;
pt.y = 0;
```

就像在其他位置一样，类型注解是可选的，如果没有指定，将是隐式的 any 类型。

字段也可以有 _初始化器_；它们会在类实例化时自动运行：

```ts twoslash
class Point {
  x = 0;
  y = 0;
}

const pt = new Point();
// Prints 0, 0
console.log(`${pt.x}, ${pt.y}`);
```

就像 `const`，`let` 和 `var` 一样，类属性的初始化器将用于推断其类型：

```ts twoslash
// @errors: 2322
class Point {
  x = 0;
  y = 0;
}
// ---cut---
const pt = new Point();
pt.x = "0";
```

#### `--strictPropertyInitialization`

[`strictPropertyInitialization`](/zh/tsconfig#strictPropertyInitialization) 配置项控制类字段是否需要在构造函数中初始化。

```ts twoslash
// @errors: 2564
class BadGreeter {
  name: string;
}
```

```ts twoslash
class GoodGreeter {
  name: string;

  constructor() {
    this.name = "hello";
  }
}
```

注意，需要在 _构造函数本身_ 中初始化字段。
TypeScript 不会分析你从构造函数调用的方法来检测初始化，因为派生类可能会覆盖这些方法并导致成员没有被初始化。

如果你打算通过构造函数以外的方式来初始化一个字段（例如，可能有一个外部库会为你填充类的一部分），你可以使用 _确切的赋值断言操作符_ `!`:

```ts twoslash
class OKGreeter {
  // Not initialized, but no error
  name!: string;
}
```

### `readonly`

字段前面可以加上 `readonly` 修饰符。
这可以防止在构造函数外部对字段进行赋值。

```ts twoslash
// @errors: 2540 2540
class Greeter {
  readonly name: string = "world";

  constructor(otherName?: string) {
    if (otherName !== undefined) {
      this.name = otherName;
    }
  }

  err() {
    this.name = "not ok";
  }
}
const g = new Greeter();
g.name = "also not ok";
```

### Constructors

<blockquote class='bg-reading'>
   <p>背景阅读:<br />
   <a href='https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/constructor'>Constructor (MDN)</a><br/>
   </p>
</blockquote>

类的构造函数与普通函数非常相似。
你可以添加参数，带有类型注解，默认值和重载：

```ts twoslash
class Point {
  x: number;
  y: number;

  // Normal signature with defaults
  constructor(x = 0, y = 0) {
    this.x = x;
    this.y = y;
  }
}
```

```ts twoslash
class Point {
  x: number = 0;
  y: number = 0;

  // Constructor overloads
  constructor(x: number, y: number);
  constructor(xy: string);
  constructor(x: string | number, y: number = 0) {
    // Code logic here
  }
}
```

类的构造函数签名与函数签名之间只有几个区别：

- 构造函数不能有类型参数 - 这些属于外部类声明，我们稍后会学习
- 构造函数不能有返回类型注解 - 返回的总是类实例类型

#### 调用 Super

正如在 JavaScript 中一样，如果你有一个基类，你需要在你的构造函数体中调用 `super();` 之后才能使用任何的 `this.` 成员：

```ts twoslash
// @errors: 17009
class Base {
  k = 4;
}

class Derived extends Base {
  constructor() {
    // Prints a wrong value in ES5; throws exception in ES6
    console.log(this.k);
    super();
  }
}
```

忘记调用 `super` 在 JavaScript 中是一个容易犯的错误，但 TypeScript 会告诉你什么时候需要这样做。

### Methods

<blockquote class='bg-reading'>
   <p>背景阅读:<br />
   <a href='https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Method_definitions'>Method definitions</a><br/>
   </p>
</blockquote>

类的函数属性被称为 _方法_。
方法可以使用与函数和构造函数相同的类型注解：

```ts twoslash
class Point {
  x = 10;
  y = 10;

  scale(n: number): void {
    this.x *= n;
    this.y *= n;
  }
}
```

除了标准的类型注解，TypeScript 对方法没有添加任何其他新的内容。

请注意，在方法体内部，访问字段和其他方法都必须通过 `this.`。
在方法体中的未说明名称将始终指的是包围范围的内容：

```ts twoslash
// @errors: 2322
let x: number = 0;

class C {
  x: string = "hello";

  m() {
    // This is trying to modify 'x' from line 1, not the class property
    x = "world";
  }
}
```

### Getters / Setters

类也可以拥有 _访问器_：

```ts twoslash
class C {
  _length = 0;
  get length() {
    return this._length;
  }
  set length(value) {
    this._length = value;
  }
}
```

> 注意，很少有情况下，一个没有额外逻辑的由字段支持的 get/set 对是在 JavaScript 中有用的。
> 如果你在 get/set 操作期间不需要添加额外的逻辑，那么公开公共字段是可以的。

TypeScript 对访问器有一些特殊的推断规则：

- 如果存在 `get` 但没有 `set`，属性自动为 `readonly`
- 如果未指定 setter 参数的类型，它将从 getter 的返回类型中推断

从 [TypeScript 4.3](https://devblogs.microsoft.com/typescript/announcing-typescript-4-3/) 开始，get 和 set 的访问器可以有不同的类型。

```ts twoslash
class Thing {
  _size = 0;

  get size(): number {
    return this._size;
  }

  set size(value: string | number | boolean) {
    let num = Number(value);

    // Don't allow NaN, Infinity, etc

    if (!Number.isFinite(num)) {
      this._size = 0;
      return;
    }

    this._size = num;
  }
}
```

### 索引签名

类可以声明索引签名；这些与[其他对象类型的索引签名]((/zh/docs/handbook/2/objects.html#index-signatures))工作方式相同：

```ts twoslash
class MyClass {
  [s: string]: boolean | ((s: string) => boolean);

  check(s: string) {
    return this[s] as boolean;
  }
}
```

由于索引签名类型需要同时捕获方法的类型，因此并不容易将这些类型用于实际应用。
一般来说，最好在类实例本身之外的另一个地方存储索引数据。

## 类继承

像其他具有面向对象特性的语言一样，JavaScript 中的类可以从基类继承。

### `implements` 子句

你可以使用 `implements` 子句来检查一个类是否满足一个特定的 `interface。`
如果一个类没有正确实现，将会产生一个错误：

```ts twoslash
// @errors: 2420
interface Pingable {
  ping(): void;
}

class Sonar implements Pingable {
  ping() {
    console.log("ping!");
  }
}

class Ball implements Pingable {
  pong() {
    console.log("pong!");
  }
}
```

类也可以实现多个接口，例如 `class C implements A, B {`。

#### 注意事项

需要理解的是，`implements` 子句只是检查类可以被看作是接口类型的一个检查。
如果一个类没有正确地实现，将会产生一个错误。它 _完全_ 并不会改变类或其方法的类型。
一个常见的错误源是假设 `implements` 子句会改变类类型 - 它不会！

```ts twoslash
// @errors: 7006
interface Checkable {
  check(name: string): boolean;
}

class NameChecker implements Checkable {
  check(s) {
    // Notice no error here
    return s.toLowerCase() === "ok";
    //         ^?
  }
}
```

在这个例子中，我们可能希望 `s` 的类型会受到 `check` 中的 `name: string` 参数的影响。
然而并没有 - `implements` 子句没有改变类体的检查方式或它的类型推断。

同样，实现一个有可选属性的接口不会创建那个属性：

```ts twoslash
// @errors: 2339
interface A {
  x: number;
  y?: number;
}
class C implements A {
  x = 0;
}
const c = new C();
c.y = 10;
```

### `extends` 子句

<blockquote class='bg-reading'>
   <p>背景阅读:<br />
   <a href='https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/extends'>extends keyword (MDN)</a><br/>
   </p>
</blockquote>

类可以从基类 `extends`。
派生类拥有其基类的所有属性和方法，还可以定义其他成员。

```ts twoslash
class Animal {
  move() {
    console.log("Moving along!");
  }
}

class Dog extends Animal {
  woof(times: number) {
    for (let i = 0; i < times; i++) {
      console.log("woof!");
    }
  }
}

const d = new Dog();
// Base class method
d.move();
// Derived class method
d.woof(3);
```

#### 重写方法

<blockquote class='bg-reading'>
   <p>背景阅读:<br />
   <a href='https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/super'>super keyword (MDN)</a><br/>
   </p>
</blockquote>

派生类也可以重写基类的字段或属性。
你可以使用 `super`. 语法来访问基类的方法。
注意，由于 JavaScript 类只是一个简单的查找对象，所以没有“超级字段”的概念。

TypeScript 强制要求派生类始终是其基类的子类型。

例如，这是重写方法的合法方式：

```ts twoslash
class Base {
  greet() {
    console.log("Hello, world!");
  }
}

class Derived extends Base {
  greet(name?: string) {
    if (name === undefined) {
      super.greet();
    } else {
      console.log(`Hello, ${name.toUpperCase()}`);
    }
  }
}

const d = new Derived();
d.greet();
d.greet("reader");
```

要确保派生类遵循其基类的契约非常重要。
请记住，通过基类引用引用派生类实例是非常常见的（并且总是合法的！）：

```ts twoslash
class Base {
  greet() {
    console.log("Hello, world!");
  }
}
class Derived extends Base {}
const d = new Derived();
// ---cut---
// Alias the derived instance through a base class reference
const b: Base = d;
// No problem
b.greet();
```

如果 `Derived` 没有遵循 `Base` 的契约，会发生什么呢？

```ts twoslash
// @errors: 2416
class Base {
  greet() {
    console.log("Hello, world!");
  }
}

class Derived extends Base {
  // Make this parameter required
  greet(name: string) {
    console.log(`Hello, ${name.toUpperCase()}`);
  }
}
```

如果我们无视错误继续编译这段代码，这样的示例代码会崩溃：

```ts twoslash
declare class Base {
  greet(): void;
}
declare class Derived extends Base {}
// ---cut---
const b: Base = new Derived();
// Crashes because "name" will be undefined
b.greet();
```

#### 仅限类型的字段声明

当 `target >= ES2022` 或 [`useDefineForClassFields`](/zh/tsconfig#useDefineForClassFields) 设置为 true 时，类字段在父类构造函数完成后被初始化，覆盖由父类设置的任何值。这在你只想重新声明一个继承字段更准确的类型时可能会是一个问题。为了处理这种情况，你可以编写 `declare` 来告诉 TypeScript 这个字段声明不应有任何运行时影响。

```ts twoslash
interface Animal {
  dateOfBirth: any;
}

interface Dog extends Animal {
  breed: any;
}

class AnimalHouse {
  resident: Animal;
  constructor(animal: Animal) {
    this.resident = animal;
  }
}

class DogHouse extends AnimalHouse {
  // Does not emit JavaScript code,
  // only ensures the types are correct
  declare resident: Dog;
  constructor(dog: Dog) {
    super(dog);
  }
}
```

#### 初始化顺序

JavaScript 类的初始化顺序在某些情况下可能令人感到惊奇。
我们来考虑下面这段代码：

```ts twoslash
class Base {
  name = "base";
  constructor() {
    console.log("My name is " + this.name);
  }
}

class Derived extends Base {
  name = "derived";
}

// Prints "base", not "derived"
const d = new Derived();
```

这是怎么回事呢？

如JavaScript所定义，类的初始化顺序是这样的：

- 初始化基类字段
- 执行基类构造函数
- 初始化派生类字段
- 执行派生类构造函数

这意味着基类构造函数在其自身构造函数中看到的是其自己的 name 值，因为此时派生类字段的初始化还没有执行。

#### 继承内建类型

> 注意：如果你没有计划继承像 `Array`、`Error`、`Map` 等内建类型，或者你的编译目标明确设为 `ES6`/`ES2015` 或以上，你可以跳过这一部分

在 ES2015 中，如果构造函数返回一个对象，那么 `super(...)` 的调用者会隐式地用 `this` 替换返回值。
生成的构造函数代码需要捕获 `super(...)` 的任何可能的返回值，并用 `this` 替换。

因此，继承 `Error`、`Array` 等可能不再按预期工作。
这是因为 `Error`、`Array` 等的构造函数使用 ECMAScript 6 的 `new.target` 来调整原型链；
然而，在 ECMAScript 5 中调用构造函数时，没有办法确保 `new.target` 的值。
其他向下兼容的编译器默认情况下通常也有相同的限制。

对于如下的子类：

```ts twoslash
class MsgError extends Error {
  constructor(m: string) {
    super(m);
  }
  sayHello() {
    return "hello " + this.message;
  }
}
```

你可能会发现：

- 在通过构造这些子类返回的对象上，方法可能是 `undefined`，因此调用 `sayHello` 将导致错误。
- 子类和其实例之间的 `instanceof` 会失效，因此 `(new MsgError()) instanceof MsgError` 将返回 `false`。

作为推荐，你可以在任何 `super(...)` 调用后立即手动调整原型。

```ts twoslash
class MsgError extends Error {
  constructor(m: string) {
    super(m);

    // Set the prototype explicitly.
    Object.setPrototypeOf(this, MsgError.prototype);
  }

  sayHello() {
    return "hello " + this.message;
  }
}
```

然而，任何 `MsgError` 的子类都将需要手动设置原型。
对于不支持 [`Object.setPrototypeOf`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/setPrototypeOf) 的运行时，你可能可以使用 [`__proto__`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/proto)。

遗憾的是，[这些解决方法在IE10以及更早版本的浏览器上将无法正常工作]((<https://msdn.microsoft.com/en-us/library/s4esdbwz(v=vs.94).aspx>))。
可以手动从原型上将方法复制到实例自身（即从 `MsgError.prototype` 到 `this`），但是不能修复原型链本身。

## 成员是否可见

你可以使用TypeScript来控制某些方法或属性是否对类以外的代码可见。

### `public`

类成员的默认可见性为 `public`。
`public` 成员在任何地方都可以访问：

```ts twoslash
class Greeter {
  public greet() {
    console.log("hi!");
  }
}
const g = new Greeter();
g.greet();
```

因为 `public` 已经是默认的可见性修饰符，所以你永远不需要在类成员上写 `public`，但是为了样式/可读性的原因你可能会选择这么做。

### `protected`

`protected` 成员仅对它们所在类的子类可见。

```ts twoslash
// @errors: 2445
class Greeter {
  public greet() {
    console.log("Hello, " + this.getName());
  }
  protected getName() {
    return "hi";
  }
}

class SpecialGreeter extends Greeter {
  public howdy() {
    // OK to access protected member here
    console.log("Howdy, " + this.getName());
    //                          ^^^^^^^^^^^^^^
  }
}
const g = new SpecialGreeter();
g.greet(); // OK
g.getName();
```

#### 暴露 `protected` 成员

派生类需要遵循基类的契约，但可能选择公开一个子类型的基类以提供更多的功能。
这包括将 `protected` 成员标记为 `public`：

```ts twoslash
class Base {
  protected m = 10;
}
class Derived extends Base {
  // No modifier, so default is 'public'
  m = 15;
}
const d = new Derived();
console.log(d.m); // OK
```

请注意，`Derived` 已经可以自由地读取和写入 `m`，所以这并没实质性改变这种情况的"安全"性质。
这里需要注意的主要事情是，在派生类中，如果这种暴露行为不是故意要做的，我们需要小心地重复 `protected` 修饰符。

#### 跨层级 `protected` 访问

不同的面向对象语言对是否可以通过基类引用访问 `protected` 成员有不同的意见：

```ts twoslash
// @errors: 2446
class Base {
  protected x: number = 1;
}
class Derived1 extends Base {
  protected x: number = 5;
}
class Derived2 extends Base {
  f1(other: Derived2) {
    other.x = 10;
  }
  f2(other: Derived1) {
    other.x = 10;
  }
}
```

例如，Java 认为这是合法的。
另一方面，C# 和 C++ 认为这段代码应该是非法的。

TypeScript在这里支持 C# 和 C++，因为在 `Derived2` 中访问 `x` 只能从 `Derived2` 的子类中合法地进行，而 `Derived1` 不是其中之一。
此外，如果通过 `Derived1` 引用访问 `x` 是非法的（它当然应该是！），那么通过基类引用访问不应改善这种情况。

参考 [为什么我不能在派生的类中访问 protected 成员呢？]((https://blogs.msdn.microsoft.com/ericlippert/2005/11/09/why-cant-i-access-a-protected-member-from-a-derived-class/)) 这篇文章可以获取更多的 C# 的理由。

### `private`

`private` 类似于 `protected`，但即使是子类也无法访问这个成员：

```ts twoslash
// @errors: 2341
class Base {
  private x = 0;
}
const b = new Base();
// Can't access from outside the class
console.log(b.x);
```

```ts twoslash
// @errors: 2341
class Base {
  private x = 0;
}
// ---cut---
class Derived extends Base {
  showX() {
    // Can't access in subclasses
    console.log(this.x);
  }
}
```

因为 `private` 成员对派生类来说是不可见的，所以派生类无法增加其可见性：

```ts twoslash
// @errors: 2415
class Base {
  private x = 0;
}
class Derived extends Base {
  x = 1;
}
```

#### 跨实例 private 访问

不同的面向对象语言对同一类的不同实例是否可以访问彼此的 `private` 成员有不同的观点。
虽然像 Java，C#，C++，Swift 和 PHP 这样的语言允许这样做，但 Ruby 不允许。

TypeScript 允许跨实例 `private` 访问：

```ts twoslash
class A {
  private x = 10;

  public sameAs(other: A) {
    // No error
    return other.x === this.x;
  }
}
```

#### 注意事项

像 TypeScript 的类型系统的其他方面一样，`private` 和 `protected` [只在类型检查期间执行](https://www.doc.tslang.org/zh/play?removeComments=true&target=99&ts=4.3.4#code/PTAEGMBsEMGddAEQPYHNQBMCmVoCcsEAHPASwDdoAXLUAM1K0gwQFdZSA7dAKWkoDK4MkSoByBAGJQJLAwAeAWABQIUH0HDSoiTLKUaoUggAW+DHorUsAOlABJcQlhUy4KpACeoLJzrI8cCwMGxU1ABVPIiwhESpMZEJQTmR4lxFQaQxWMm4IZABbIlIYKlJkTlDlXHgkNFAAbxVQTIAjfABrAEEC5FZOeIBeUAAGAG5mmSw8WAroSFIqb2GAIjMiIk8VieVJ8Ar01ncAgAoASkaAXxVr3dUwGoQAYWpMHBgCYn1rekZmNg4eUi0Vi2icoBWJCsNBWoA6WE8AHcAiEwmBgTEtDovtDaMZQLM6PEoQZbA5wSk0q5SO4vD4-AEghZoJwLGYEIRwNBoqAzFRwCZCFUIlFMXECdSiAhId8YZgclx0PsiiVqOVOAAaUAFLAsxWgKiC35MFigfC0FKgSAVVDTSyk+W5dB4fplHVVR6gF7xJrKFotEk-HXIRE9PoDUDDcaTAPTWaceaLZYQlmoPBbHYx-KcQ7HPDnK43FQqfY5+IMDDISPJLCIuqoc47UsuUCofAME3Vzi1r3URvF5QV5A2STtPDdXqunZDgDaYlHnTDrrEAF0dm28B3mDZg6HJwN1+2-hg57ulwNV2NQGoZbjYfNrYiENBwEFaojFiZQK08C-4fFKTVCozWfTgfFgLkeT5AUqiAA).

这意味着像 `in` 这样的 JavaScript 运行时结构或简单属性查找仍然可以访问 `private` 或 `protected` 成员：

```ts twoslash
class MySafe {
  private secretKey = 12345;
}
```

```js
// In a JavaScript file...
const s = new MySafe();
// Will print 12345
console.log(s.secretKey);
```

`private` 还允许在类型检查期间使用方括号表示法访问。这使得声明为 `private` 的字段可能更易于访问，例如在单元测试等场景下，但问题是这些字段是 _软私有_ 的，不严格的强制执行隐私。

```ts twoslash
// @errors: 2341
class MySafe {
  private secretKey = 12345;
}

const s = new MySafe();

// Not allowed during type checking
console.log(s.secretKey);

// OK
console.log(s["secretKey"]);
```

与 TypeScript 的 `private` 不同，JavaScript 的 [私有字段]((https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/Private_class_fields))（`#`）在编译后仍然是私有的，不提供先前提到的逃逸舱，比如通过方括号表示法来访问，这使得它们成为 _硬私有_。

```ts twoslash
class Dog {
  #barkAmount = 0;
  personality = "happy";

  constructor() {}
}
```

```ts twoslash
// @target: esnext
// @showEmit
class Dog {
  #barkAmount = 0;
  personality = "happy";

  constructor() {}
}
```

当编译到 ES2021 或更低版本时，TypeScript 将使用 WeakMaps 代替 `#`。

```ts twoslash
// @target: es2015
// @showEmit
class Dog {
  #barkAmount = 0;
  personality = "happy";

  constructor() {}
}
```

如果你需要在类中保护一些值，避免恶意行为，你应该使用提供硬运行时隐私性的机制，如闭包，WeakMaps，或者私有字段。请注意，这些在运行时的额外隐私性检查可能会影响性能。

## 静态（static）成员

<blockquote class='bg-reading'>
   <p>背景阅读:<br />
   <a href='https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/static'>Static Members (MDN)</a><br/>
   </p>
</blockquote>

类可能有 静态（`static`）成员。
这些成员并不与类的具体实例有关。
他们可以通过类的构造对象本身来访问：

```ts twoslash
class MyClass {
  static x = 0;
  static printX() {
    console.log(MyClass.x);
  }
}
console.log(MyClass.x);
MyClass.printX();
```

静态成员也可以使用相同的 `public`，`protected` 和 `private` 可见性修饰符：

```ts twoslash
// @errors: 2341
class MyClass {
  private static x = 0;
}
console.log(MyClass.x);
```

静态成员也是可以被继承的：

```ts twoslash
class Base {
  static getGreeting() {
    return "Hello world";
  }
}
class Derived extends Base {
  myGreeting = Derived.getGreeting();
}
```

### 特殊的静态名称

重写来自 `Function` 原型的属性不安全也无法重写。
因为类本身就是可以用 `new` 调用的函数，所以某些 `static` 的名称是不能使用的。
函数属性如 `name`，`length` 和 `call` 不能定义成 `static` 成员：

```ts twoslash
// @errors: 2699
class S {
  static name = "S!";
}
```

### 为什么没有静态类？

TypeScript （和 JavaScript）并没有像 C# 同样的 `static class` 构造。

这些构造 _只_ 是因为那些语言强制所有数据和函数必需在类内部；由于 TypeScript 并没有这种限制，
所以没有它们的需求。在 JavaScript/TypeScript 中，只有单个实例的类通常被表示为常规的 _对象_。

例如，我们不需要在 TypeScript 中的 "静态类" 语法，因为一个常规的对象（或者甚至顶级函数）将做得同样好：

```ts twoslash
// Unnecessary "static" class
class MyStaticClass {
  static doSomething() {}
}

// Preferred (alternative 1)
function doSomething() {}

// Preferred (alternative 2)
const MyHelperObject = {
  dosomething() {},
};
```

## 类中的 `static` 块

静态块允许你编写一系列具有自己作用域的语句，这些语句可以访问包含类中的私有字段。这意味着我们可以写具有写语句所有功能，没有变量泄露，并且可以完全访问我们类的内部的初始化代码。

```ts twoslash
declare function loadLastInstances(): any[]
// ---cut---
class Foo {
    static #count = 0;

    get count() {
        return Foo.#count;
    }

    static {
        try {
            const lastInstances = loadLastInstances();
            Foo.#count += lastInstances.length;
        }
        catch {}
    }
}
```

## 泛型类

类，像接口一样，可以是泛型的。
当通过 `new` 实例化一个泛型类时，其类型参数的推断方式与函数调用相同：

```ts twoslash
class Box<Type> {
  contents: Type;
  constructor(value: Type) {
    this.contents = value;
  }
}

const b = new Box("hello!");
//    ^?
```

类可以像接口一样使用泛型约束和默认值。

### 静态成员中的类型参数

这段代码是非法的，可能不是那么明显：

```ts twoslash
// @errors: 2302
class Box<Type> {
  static defaultValue: Type;
}
```

记住类型总是完全擦除！
在运行时，只有 _一个_  `Box.defaultValue` 属性槽。
这意味着，设置 `Box<string>.defaultValue` （如果可能的话）也会改变 `Box<number>.defaultValue` - 那不好。
泛型类的 `static` 成员永远不能引用类的类型参数。

## 类运行时的的 this

<blockquote class='bg-reading'>
   <p>背景阅读:<br />
   <a href='https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/this'>this keyword (MDN)</a><br/>
   </p>
</blockquote>

重要的是要记住 TypeScript 并没有改变 JavaScript 的运行时行为，而 JavaScript 一些特殊的运行时行为声名在外。

JavaScript 对 `this` 的处理的确是非常特殊：

```ts twoslash
class MyClass {
  name = "MyClass";
  getName() {
    return this.name;
  }
}
const c = new MyClass();
const obj = {
  name: "obj",
  getName: c.getName,
};

// Prints "obj", not "MyClass"
console.log(obj.getName());
```

简而言之，默认情况下，函数内部的 `this` 值取决于 函数是如何被调用的。
在这个例子中，因为函数是通过 `obj` 引用调用的，它的 `this` 值是 `obj` 而不是类实例。

这通常不是你想要的结果！
TypeScript 提供了一些方法来减轻或防止这种错误。

### 箭头函数

<blockquote class='bg-reading'>
   <p>背景阅读:<br />
   <a href='https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions'>Arrow functions (MDN)</a><br/>
   </p>
</blockquote>

如果你有一个函数经常以丢失其 `this` 上下文的方式被调用，使用箭头函数属性而不是方法定义可能是有意义的：

```ts twoslash
class MyClass {
  name = "MyClass";
  getName = () => {
    return this.name;
  };
}
const c = new MyClass();
const g = c.getName;
// Prints "MyClass" instead of crashing
console.log(g());
```

这有一些权衡：

- `this` 值在运行时保证是正确的，即使对于没有用 TypeScript 检查的代码
- 这将使用更多内存，因为每个类实例都将拥有以这种方式定义的每个函数的自己的副本
- 你不能在派生类中使用 `super.getName`，因为原型链中没有条目可以从基类方法中获取

### `this` 参数

在方法或函数定义中，名为 `this` 的初始参数在 TypeScript 中具有特殊含义。
这些参数在编译期间会被擦除：

```ts twoslash
type SomeType = any;
// ---cut---
// TypeScript input with 'this' parameter
function fn(this: SomeType, x: number) {
  /* ... */
}
```

```js
// JavaScript output
function fn(x) {
  /* ... */
}
```

TypeScript 检查带有 `this` 参数的函数调用是否使用了正确的上下文。
我们可以在方法定义中添加一个 `this` 参数来静态强制正确调用方法，而不是使用箭头函数：

```ts twoslash
// @errors: 2684
class MyClass {
  name = "MyClass";
  getName(this: MyClass) {
    return this.name;
  }
}
const c = new MyClass();
// OK
c.getName();

// Error, would crash
const g = c.getName;
console.log(g());
```

这种方法与箭头函数方法相反的权衡：

- JavaScript 调用者可能仍然会错误地使用类方法而不自知
- 只有一个函数每个类定义分配，而不是每个类实例
- 基类方法定义仍然可以通过 `super` 调用。

## `this` 类型

在类中，一种特殊的类型称为 `this`，_动态地_ 指向当前类的类型。
让我们看看这是如何有用的：

<!-- prettier-ignore -->
```ts twoslash
class Box {
  contents: string = "";
  set(value: string) {
//  ^?
    this.contents = value;
    return this;
  }
}
```

这里，TypeScript 推断 set 的返回类型为 `this`，而不是 `Box`。
现在让我们创建一个 `Box` 的子类：

```ts twoslash
class Box {
  contents: string = "";
  set(value: string) {
    this.contents = value;
    return this;
  }
}
// ---cut---
class ClearableBox extends Box {
  clear() {
    this.contents = "";
  }
}

const a = new ClearableBox();
const b = a.set("hello");
//    ^?
```

你还可以在参数类型注解中使用 `this`：

```ts twoslash
class Box {
  content: string = "";
  sameAs(other: this) {
    return other.content === this.content;
  }
}
```

这与写 `other: Box` 不同 -- 如果你有一个派生类，它的 `sameAs` 方法现在只接受同一派生类的其他实例：

```ts twoslash
// @errors: 2345
class Box {
  content: string = "";
  sameAs(other: this) {
    return other.content === this.content;
  }
}

class DerivedBox extends Box {
  otherContent: string = "?";
}

const base = new Box();
const derived = new DerivedBox();
derived.sameAs(base);
```

### 基于 `this` 的类型守卫

你可以在类和接口的方法中使用 `this is Type` 在返回位置。
当与类型缩小（例如 `if` 语句）混合时，目标对象的类型将被缩小到指定的 `Type`。

<!-- prettier-ignore -->
```ts twoslash
// @strictPropertyInitialization: false
class FileSystemObject {
  isFile(): this is FileRep {
    return this instanceof FileRep;
  }
  isDirectory(): this is Directory {
    return this instanceof Directory;
  }
  isNetworked(): this is Networked & this {
    return this.networked;
  }
  constructor(public path: string, private networked: boolean) {}
}

class FileRep extends FileSystemObject {
  constructor(path: string, public content: string) {
    super(path, false);
  }
}

class Directory extends FileSystemObject {
  children: FileSystemObject[];
}

interface Networked {
  host: string;
}

const fso: FileSystemObject = new FileRep("foo/bar.txt", "foo");

if (fso.isFile()) {
  fso.content;
// ^?
} else if (fso.isDirectory()) {
  fso.children;
// ^?
} else if (fso.isNetworked()) {
  fso.host;
// ^?
}
```

一个常见的基于 this 的类型守卫用例是允许对特定字段进行延迟验证。例如，这种情况在 `hasValue` 被验证为真时从 `box` 中的值中移除了一个 `undefined`：

```ts twoslash
class Box<T> {
  value?: T;

  hasValue(): this is { value: T } {
    return this.value !== undefined;
  }
}

const box = new Box<string>();
box.value = "Gameboy";

box.value;
//  ^?

if (box.hasValue()) {
  box.value;
  //  ^?
}
```

## 参数属性

TypeScript 提供了一种特殊的语法，可以将构造函数的参数转化为具有相同名称和值的类属性。
这些被称为 _参数属性_，方法是在构造函数参数前添加一个可见性修饰符 `public`、`private`、`protected` 或 `readonly`，
在此过程中生成的字段获得这些修饰符。

```ts twoslash
// @errors: 2341
class Params {
  constructor(
    public readonly x: number,
    protected y: number,
    private z: number
  ) {
    // No body necessary
  }
}
const a = new Params(1, 2, 3);
console.log(a.x);
//            ^?
console.log(a.z);
```

## 类表达式

<blockquote class='bg-reading'>
   <p>背景阅读:<br />
   <a href='https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/class'>Class expressions (MDN)</a><br/>
   </p>
</blockquote>

相较于类的声明，类的表达式非常相似。
类表达式的唯一区别是不需要名称，尽管我们可以通过它们绑定到的任何标识符来引用它们：

```ts twoslash
const someClass = class<Type> {
  content: Type;
  constructor(value: Type) {
    this.content = value;
  }
};

const m = new someClass("Hello, world");
//    ^?
```

## 构造函数签名

JavaScript 类是通过 `new` 运算符实例化的。给定一个类本身的类型，[InstanceType](/zh/docs/handbook/utility-types.html#instancetypetype) 工具类型会模拟这个操作。

```ts twoslash
class Point {
  createdAt: number;
  x: number;
  y: number
  constructor(x: number, y: number) {
    this.createdAt = Date.now()
    this.x = x;
    this.y = y;
  }
}
type PointInstance = InstanceType<typeof Point>

function moveRight(point: PointInstance) {
  point.x += 5;
}

const point = new Point(3, 4);
moveRight(point);
point.x; // => 8
```

## `abstract` 类和成员

在 TypeScript 中，类、方法、和字段可能被标记为 _abstract_。

一个 _abstract method_ 或 _abstract field_ 指的是没有提供实现的方法或字段。
这些成员必须存在于一个 _abstract class_ 中，而抽象类不能被直接实例化。

抽象类的角色是作为实现了所有抽象成员的子类的基类。
当一个类没有任何抽象成员时，它被称为 _concrete_ 或具体的。

让我们看一个例子：

```ts twoslash
// @errors: 2511
abstract class Base {
  abstract getName(): string;

  printName() {
    console.log("Hello, " + this.getName());
  }
}

const b = new Base();
```

我们不能使用 `new` 实例化 `Base`，因为它是抽象的。
相反，我们需要创建一个派生类并实现抽象成员：

```ts twoslash
abstract class Base {
  abstract getName(): string;
  printName() {}
}
// ---cut---
class Derived extends Base {
  getName() {
    return "world";
  }
}

const d = new Derived();
d.printName();
```

注意，如果我们忘记实现基类的抽象成员，我们会得到一个错误：

```ts twoslash
// @errors: 2515
abstract class Base {
  abstract getName(): string;
  printName() {}
}
// ---cut---
class Derived extends Base {
  // forgot to do anything
}
```

### 抽象构造签名

有时你可能想接受一个类的构造函数，该函数会产生一个派生自某个抽象类的类的实例。

例如，你可能想编写这样的代码:

```ts twoslash
// @errors: 2511
abstract class Base {
  abstract getName(): string;
  printName() {}
}
class Derived extends Base {
  getName() {
    return "";
  }
}
// ---cut---
function greet(ctor: typeof Base) {
  const instance = new ctor();
  instance.printName();
}
```

TypeScript 正确地告诉你，你正在试图实例化一个抽象类。
毕竟，根据 `greet` 的定义，这样的代码完全合法，尽管这会构造一个抽象类：

```ts twoslash
declare const greet: any, Base: any;
// ---cut---
// Bad!
greet(Base);
```

相反，你可能想要编写一个带有构造签名的函数：

```ts twoslash
// @errors: 2345
abstract class Base {
  abstract getName(): string;
  printName() {}
}
class Derived extends Base {
  getName() {
    return "";
  }
}
// ---cut---
function greet(ctor: new () => Base) {
  const instance = new ctor();
  instance.printName();
}
greet(Derived);
greet(Base);
```

现在 TypeScript 正确地告诉你哪个类的构造函数可以被调用 - `Derived` 可以，因为它是具体的，但 `Base` 不可以。

## 类之间的关系

在大多数情况下，TypeScript 中的类的结构是可以对比的，就像其他类型一样。

例如，这两个类可以互换使用，因为它们是完全相同的：

```ts twoslash
class Point1 {
  x = 0;
  y = 0;
}

class Point2 {
  x = 0;
  y = 0;
}

// OK
const p: Point1 = new Point2();
```

同样，类之间的子类型关系存在即使没有显式的继承：

```ts twoslash
// @strict: false
class Person {
  name: string;
  age: number;
}

class Employee {
  name: string;
  age: number;
  salary: number;
}

// OK
const p: Person = new Employee();
```

这听起来很直接，但有一些情况稍微奇特一些。

空类没有成员。
在结构化类型系统中，没有成员的类型通常是其他任何事物的超类型。
所以如果你写一个空类（请不要！），任何东西都可以替代它：

```ts twoslash
class Empty {}

function fn(x: Empty) {
  // can't do anything with 'x', so I won't
}

// All OK!
fn(window);
fn({});
fn(fn);
```
