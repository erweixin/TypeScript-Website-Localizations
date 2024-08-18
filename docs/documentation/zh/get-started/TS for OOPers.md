---
title: 给 Java/C# 程序员的 TypeScript 入门指南
short: 给 Java/C# 程序员的 TypeScript 入门指南
layout: docs
permalink: /zh/docs/handbook/typescript-in-5-minutes-oop.html
oneline: Learn TypeScript if you have a background in object-oriented languages
---

习惯使用静态类型语言（如 C# 和 Java）的程序员常常会选择 TypeScript。

TypeScript 的类型系统带来了许多相同的优点，例如更好的代码补全，更早发现错误，以及程序各部分间更清晰的调用等等。
尽管 TypeScript 为这些开发者提供了许多熟悉的功能，但还是值得退后一步看看 JavaScript（以及因此而产生的 TypeScript）与传统的面向对象编程语言有何不同。
理解这些差异将有助于你编写更好的 JavaScript 代码，并规避从 C#/Java 切换到 TypeScript 的常见误区。

## 学习 JavaScript

如果你已经熟悉 JavaScript，但主要是使用 Java 或 C# 进行编程，那么这个入门页面将有助于解释一些你可能容易误解和陷入的问题。
TypeScript 对类型的模型化方式与 Java 或 C# 有很大的不同，所以在学习 TypeScript 时，请记住这些不同之处。

如果你是 Java 或 C# 程序员，对 JavaScript 基本上还是新手，我们推荐你先学习一点点 _无_ 类型的 JavaScript，以理解 JavaScript 的运行时行为。
因为 TypeScript 并不改变你的代码的 _运行方式_，所以你还是得学习 JavaScript 的工作原理，才能编写真正有实际效果的代码！

记住，TypeScript 使用的是和 JavaScript 相同的_运行时_，所以任何关于如何实现特定运行时行为的资源（如将字符串转换为数字，显示警告，将文件写入到磁盘等等）都同样适用于 TypeScript 程序。
不要仅限于只使用针对 TypeScript 的学习资源！

## 重新审视 Class

C# 和 Java 是我们所称的 _强制 OOP_ 语言。
在这些语言中，class 是代码组织的基本单位，同时也是运行时所有数据 _和_ 行为的基本容器。
强制所有的功能和数据都要放在类中，可能是解决某些问题的好模型，但并不是每个领域 _都需要_ 用这种方式来表示。

### 自由的函数和数据

在 JavaScript 中，函数可以生存于任何地方，数据可以自由地传递，而不需要被包含在预先定义的 `class` 或 `struct` 中。
这种灵活性是极其有力的。
"自由"的函数（那些与类无关的函数）会在没有隐含的 OOP 层次结构的情况下处理数据，这倾向于成为编写 JavaScript 程序的首选模型。

### 静态类

此外，C# 和 Java 中的一些构造，如单例和静态类，在 TypeScript 中并不需要。

## 在 TypeScript 中的 OOP

话又说回来，如果你喜欢的话，你仍然可以使用类！
一些问题很适合用传统的 OOP 层次结构来解决，而 TypeScript 对 JavaScript 类的支持则会使这些模型更加强大。
TypeScript 支持许多常见的模式，如实现接口，继承，以及静态方法。

我们会在本指南后面的章节中讲解类。

## 重新思考类型

TypeScript 对 _类型_ 的理解实际上与 C# 或 Java 非常不同。
我们来探讨一些差异。



### 固定化的名义化类型系统

In C# or Java, any given value or object has one exact type - either `null`, a primitive, or a known class type.
We can call methods like `value.GetType()` or `value.getClass()` to query the exact type at runtime.
The definition of this type will reside in a class somewhere with some name, and we can't use two classes with similar shapes in lieu of each other unless there's an explicit inheritance relationship or commonly-implemented interface.

These aspects describe a _reified, nominal_ type system.
The types we wrote in the code are present at runtime, and the types are related via their declarations, not their structures.

### Types as Sets

In C# or Java, it's meaningful to think of a one-to-one correspondence between runtime types and their compile-time declarations.

In TypeScript, it's better to think of a type as a _set of values_ that share something in common.
Because types are just sets, a particular value can belong to _many_ sets at the same time.

Once you start thinking of types as sets, certain operations become very natural.
For example, in C#, it's awkward to pass around a value that is _either_ a `string` or `int`, because there isn't a single type that represents this sort of value.

In TypeScript, this becomes very natural once you realize that every type is just a set.
How do you describe a value that either belongs in the `string` set or the `number` set?
It simply belongs to the _union_ of those sets: `string | number`.

TypeScript provides a number of mechanisms to work with types in a set-theoretic way, and you'll find them more intuitive if you think of types as sets.

### Erased Structural Types

In TypeScript, objects are _not_ of a single exact type.
For example, if we construct an object that satisfies an interface, we can use that object where that interface is expected even though there was no declarative relationship between the two.

```ts twoslash
interface Pointlike {
  x: number;
  y: number;
}
interface Named {
  name: string;
}

function logPoint(point: Pointlike) {
  console.log("x = " + point.x + ", y = " + point.y);
}

function logName(x: Named) {
  console.log("Hello, " + x.name);
}

const obj = {
  x: 0,
  y: 0,
  name: "Origin",
};

logPoint(obj);
logName(obj);
```

TypeScript's type system is _structural_, not nominal: We can use `obj` as a `Pointlike` because it has `x` and `y` properties that are both numbers.
The relationships between types are determined by the properties they contain, not whether they were declared with some particular relationship.

TypeScript's type system is also _not reified_: There's nothing at runtime that will tell us that `obj` is `Pointlike`.
In fact, the `Pointlike` type is not present _in any form_ at runtime.

Going back to the idea of _types as sets_, we can think of `obj` as being a member of both the `Pointlike` set of values and the `Named` set of values.

### Consequences of Structural Typing

OOP programmers are often surprised by two particular aspects of structural typing.

#### Empty Types

The first is that the _empty type_ seems to defy expectation:

```ts twoslash
class Empty {}

function fn(arg: Empty) {
  // do something?
}

// No error, but this isn't an 'Empty' ?
fn({ k: 10 });
```

TypeScript determines if the call to `fn` here is valid by seeing if the provided argument is a valid `Empty`.
It does so by examining the _structure_ of `{ k: 10 }` and `class Empty { }`.
We can see that `{ k: 10 }` has _all_ of the properties that `Empty` does, because `Empty` has no properties.
Therefore, this is a valid call!

This may seem surprising, but it's ultimately a very similar relationship to one enforced in nominal OOP languages.
A subclass cannot _remove_ a property of its base class, because doing so would destroy the natural subtype relationship between the derived class and its base.
Structural type systems simply identify this relationship implicitly by describing subtypes in terms of having properties of compatible types.

#### Identical Types

Another frequent source of surprise comes with identical types:

```ts
class Car {
  drive() {
    // hit the gas
  }
}
class Golfer {
  drive() {
    // hit the ball far
  }
}

// No error?
let w: Car = new Golfer();
```

Again, this isn't an error because the _structures_ of these classes are the same.
While this may seem like a potential source of confusion, in practice, identical classes that shouldn't be related are not common.

We'll learn more about how classes relate to each other in the Classes chapter.

### Reflection

OOP programmers are accustomed to being able to query the type of any value, even a generic one:

```csharp
// C#
static void LogType<T>() {
    Console.WriteLine(typeof(T).Name);
}
```

Because TypeScript's type system is fully erased, information about e.g. the instantiation of a generic type parameter is not available at runtime.

JavaScript does have some limited primitives like `typeof` and `instanceof`, but remember that these operators are still working on the values as they exist in the type-erased output code.
For example, `typeof (new Car())` will be `"object"`, not `Car` or `"Car"`.

## Next Steps

This was a brief overview of the syntax and tools used in everyday TypeScript. From here, you can:

- Read the full Handbook [from start to finish](/docs/handbook/intro.html)
- Explore the [Playground examples](/play#show-examples)
