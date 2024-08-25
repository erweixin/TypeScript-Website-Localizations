---
title: 条件类型
layout: docs
permalink: /zh/docs/handbook/2/conditional-types.html
oneline: "创建在类型系统中如同 if 语句一样工作的类型。"
---

在大多数有用的程序核心中，我们需要基于输入做出决策。
JavaScript 程序也不例外，但考虑到可以轻松检查值的类型，这些决策也基于输入的类型。
_条件类型_ 有助于描述输入类型和输出类型之间的关系。

```ts twoslash
interface Animal {
  live(): void;
}
interface Dog extends Animal {
  woof(): void;
}

type Example1 = Dog extends Animal ? number : string;
//   ^?

type Example2 = RegExp extends Animal ? number : string;
//   ^?
```

条件类型的形式看起来有点像 JavaScript 中的条件表达式（`condition ? trueExpression : falseExpression`）：

```ts twoslash
type SomeType = any;
type OtherType = any;
type TrueType = any;
type FalseType = any;
type Stuff =
  // ---cut---
  SomeType extends OtherType ? TrueType : FalseType;
```

当 `extends` 左侧的类型可以赋值给右侧的类型时，你会得到第一个分支中的类型（"true" 分支）；否则你会得到后一个分支中的类型（"false" 分支）。

从上面的例子中，条件类型可能看起来并不立即有用 —— 我们可以自己判断 `Dog extends Animal` 是否成立，然后选择 `number` 或 `string`！
但条件类型的强大之处在于将它们与泛型一起使用。

例如，让我们看看下面的 `createLabel` 函数：

```ts twoslash
interface IdLabel {
  id: number /* some fields */;
}
interface NameLabel {
  name: string /* other fields */;
}

function createLabel(id: number): IdLabel;
function createLabel(name: string): NameLabel;
function createLabel(nameOrId: string | number): IdLabel | NameLabel;
function createLabel(nameOrId: string | number): IdLabel | NameLabel {
  throw "unimplemented";
}
```

这些 `createLabel` 的重载描述了一个单一的 JavaScript 函数，它根据输入的类型做出选择。注意以下几点：

1. 如果一个库需要在其整个 API 中反复做出同样的选择，这会变得很麻烦。
2. 我们必须创建三个重载：两个用于我们确定类型的情况（一个用于 `string`，一个用于 `number`），还有一个用于最一般的情况（接受 `string | number`）。对于 createLabel 可以处理的每一个新类型，重载的数量都会呈指数增长。

相反，我们可以将这种逻辑编码到一个条件类型中：

```ts twoslash
interface IdLabel {
  id: number /* some fields */;
}
interface NameLabel {
  name: string /* other fields */;
}
// ---cut---
type NameOrId<T extends number | string> = T extends number
  ? IdLabel
  : NameLabel;
```

然后我们可以使用这个条件类型来简化我们的重载，将其减少为一个没有重载的单一函数。

```ts twoslash
interface IdLabel {
  id: number /* some fields */;
}
interface NameLabel {
  name: string /* other fields */;
}
type NameOrId<T extends number | string> = T extends number
  ? IdLabel
  : NameLabel;
// ---cut---
function createLabel<T extends number | string>(idOrName: T): NameOrId<T> {
  throw "unimplemented";
}

let a = createLabel("typescript");
//  ^?

let b = createLabel(2.8);
//  ^?

let c = createLabel(Math.random() ? "hello" : 42);
//  ^?
```

### 条件类型约束

通常，条件类型中的检查会为我们提供一些新信息。
就像使用类型守卫进行类型收窄可以给我们更具体的类型一样，条件类型的 true 分支会通过我们检查的类型进一步约束泛型。

例如，让我们看看下面的例子：

```ts twoslash
// @errors: 2536
type MessageOf<T> = T["message"];
```

在这个例子中，TypeScript 报错是因为 `T` 不知道是否有一个叫做 `message` 的属性。
我们可以约束 `T`，这样 TypeScript 就不会再抱怨：

```ts twoslash
type MessageOf<T extends { message: unknown }> = T["message"];

interface Email {
  message: string;
}

type EmailMessageContents = MessageOf<Email>;
//   ^?
```

然而，如果我们想让 `MessageOf` 接受任何类型，并在没有 `message` 属性时默认为类似 `never` 的东西呢？
我们可以通过将约束移出并引入一个条件类型来做到这一点：

```ts twoslash
type MessageOf<T> = T extends { message: unknown } ? T["message"] : never;

interface Email {
  message: string;
}

interface Dog {
  bark(): void;
}

type EmailMessageContents = MessageOf<Email>;
//   ^?

type DogMessageContents = MessageOf<Dog>;
//   ^?
```

在 true 分支中，TypeScript 知道 `T` 将有一个 `message` 属性。

作为另一个例子，我们也可以写一个叫做 `Flatten` 的类型，它可以将数组类型平铺成它们的元素类型，但对其他类型则保持不变：

```ts twoslash
type Flatten<T> = T extends any[] ? T[number] : T;

// Extracts out the element type.
type Str = Flatten<string[]>;
//   ^?

// Leaves the type alone.
type Num = Flatten<number>;
//   ^?
```

当 `Flatten` 被给定一个数组类型时，它使用带有 `number` 的索引访问来获取 `string[]` 的元素类型。
否则，它只是返回给定的类型。

### 在条件类型中进行推断

我们刚刚发现自己使用条件类型来应用约束，然后提取出类型。
这最终成为一个如此常见的操作，以至于条件类型使其变得更容易。

条件类型为我们提供了一种方式，可以使用 `infer` 关键字从我们在 true 分支中比较的类型中进行推断。
例如，我们可以在 `Flatten` 中推断元素类型，而不是用索引访问类型"手动"获取它：

```ts twoslash
type Flatten<Type> = Type extends Array<infer Item> ? Item : Type;
```

在这里，我们使用 `infer` 关键字以声明的方式引入一个名为 `Item` 的新泛型类型变量，而不是在 true 分支中指定如何检索 `Type` 的元素类型。
这使我们不必考虑如何深入挖掘和探索我们感兴趣的类型的结构。

我们可以使用 `infer` 关键字编写一些有用的辅助类型别名。
例如，对于简单的情况，我们可以从函数类型中提取返回类型：

```ts twoslash
type GetReturnType<Type> = Type extends (...args: never[]) => infer Return
  ? Return
  : never;

type Num = GetReturnType<() => number>;
//   ^?

type Str = GetReturnType<(x: string) => string>;
//   ^?

type Bools = GetReturnType<(a: boolean, b: boolean) => boolean[]>;
//   ^?
```

当从具有多个调用签名的类型（如重载函数的类型）进行推断时，推断是从 _最后一个_ 签名进行的（这可能是最宽松的 catch-all 情况）。无法基于参数类型列表执行重载解析。

```ts twoslash
declare function stringOrNum(x: string): number;
declare function stringOrNum(x: number): string;
declare function stringOrNum(x: string | number): string | number;

type T1 = ReturnType<typeof stringOrNum>;
//   ^?
```

## 分布式条件类型

当条件类型作用于泛型类型时，如果给定一个联合类型，它们就会变成 _分布式_的。
例如，看看下面的例子：

```ts twoslash
type ToArray<Type> = Type extends any ? Type[] : never;
```

如果我们将一个联合类型传入 `ToArray`，则条件类型将应用于该联合的每个成员。

```ts twoslash
type ToArray<Type> = Type extends any ? Type[] : never;

type StrArrOrNumArr = ToArray<string | number>;
//   ^?
```

这里发生的是 `ToArray` 分布在：

```ts twoslash
type StrArrOrNumArr =
  // ---cut---
  string | number;
```

并映射到联合的每个成员类型上，相当于：

```ts twoslash
type ToArray<Type> = Type extends any ? Type[] : never;
type StrArrOrNumArr =
  // ---cut---
  ToArray<string> | ToArray<number>;
```

最终得到：

```ts twoslash
type StrArrOrNumArr =
  // ---cut---
  string[] | number[];
```

通常，分布性是期望的行为。
要避免这种行为，你可以用方括号将 `extends` 关键字的每一侧括起来。

```ts twoslash
type ToArrayNonDist<Type> = [Type] extends [any] ? Type[] : never;

// 'ArrOfStrOrNum' is no longer a union.
type ArrOfStrOrNum = ToArrayNonDist<string | number>;
//   ^?
```
