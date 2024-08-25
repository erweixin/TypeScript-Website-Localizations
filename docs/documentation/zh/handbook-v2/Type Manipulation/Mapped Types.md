---
title: 映射类型
layout: docs
permalink: /zh/docs/handbook/2/mapped-types.html
oneline: "通过重用现有类型生成新类型。"
---

当你不想重复自己时，有时一个类型需要基于另一个类型。

映射类型建立在索引签名的语法之上，这些语法用于声明未提前声明的属性的类型：

```ts twoslash
type Horse = {};
// ---cut---
type OnlyBoolsAndHorses = {
  [key: string]: boolean | Horse;
};

const conforms: OnlyBoolsAndHorses = {
  del: true,
  rodney: false,
};
```

映射类型是一种泛型类型，它使用 `PropertyKey` 的并集（通常通过 [`keyof`](/zh/docs/handbook/2/indexed-access-types.html)创建）来遍历键以创建类型：

```ts twoslash
type OptionsFlags<Type> = {
  [Property in keyof Type]: boolean;
};
```

在这个例子中，`OptionsFlags` 会从 `Type` 类型中获取所有的属性，并将其值更改为布尔值。

```ts twoslash
type OptionsFlags<Type> = {
  [Property in keyof Type]: boolean;
};
// ---cut---
type Features = {
  darkMode: () => void;
  newUserProfile: () => void;
};

type FeatureOptions = OptionsFlags<Features>;
//   ^?
```

### 映射修饰符


在映射过程中可以应用两个额外的修饰符：`readonly` 和 `?`，分别影响可变性和可选性。

你可以通过使用`-` 或 `+` 添加或删除这些修饰符。如果你不添加前缀，那么会默认添加 `+`。

```ts twoslash
// Removes 'readonly' attributes from a type's properties
type CreateMutable<Type> = {
  -readonly [Property in keyof Type]: Type[Property];
};

type LockedAccount = {
  readonly id: string;
  readonly name: string;
};

type UnlockedAccount = CreateMutable<LockedAccount>;
//   ^?
```

```ts twoslash
// Removes 'optional' attributes from a type's properties
type Concrete<Type> = {
  [Property in keyof Type]-?: Type[Property];
};

type MaybeUser = {
  id: string;
  name?: string;
  age?: number;
};

type User = Concrete<MaybeUser>;
//   ^?
```

## 使用 `as` 进行键重映射

在 TypeScript 4.1 及以后的版本中，你可以在映射类型中使用 `as` 子句来对键进行重映射：

```ts
type MappedTypeWithNewProperties<Type> = {
    [Properties in keyof Type as NewKeyType]: Type[Properties]
}
```

你可以利用 [模板自变量类型](/zh/docs/handbook/2/template-literal-types.html) 的特性来从旧的属性名称创建新的属性名称：

```ts twoslash
type Getters<Type> = {
    [Property in keyof Type as `get${Capitalize<string & Property>}`]: () => Type[Property]
};

interface Person {
    name: string;
    age: number;
    location: string;
}

type LazyPerson = Getters<Person>;
//   ^?
```

你可以通过产生 never 的条件类型来过滤掉键：

```ts twoslash
// Remove the 'kind' property
type RemoveKindField<Type> = {
    [Property in keyof Type as Exclude<Property, "kind">]: Type[Property]
};

interface Circle {
    kind: "circle";
    radius: number;
}

type KindlessCircle = RemoveKindField<Circle>;
//   ^?
```

你可以对任意的并集进行映射，不仅仅是 `string | number | symbol` 的并集，而且可以是任何类型的并集：



```ts twoslash
type EventConfig<Events extends { kind: string }> = {
    [E in Events as E["kind"]]: (event: E) => void;
}

type SquareEvent = { kind: "square", x: number, y: number };
type CircleEvent = { kind: "circle", radius: number };

type Config = EventConfig<SquareEvent | CircleEvent>
//   ^?
```

### 进一步探索

映射类型与本类型操作章节中的其他功能搭配得很好，例如这里是一个[使用条件类型的映射类型]((/zh/docs/handbook/2/conditional-types.html))，它根据一个对象是否拥有 `pii` 属性设置为字面值 `true` 来返回一个 `true` 或 `false`：

```ts twoslash
type ExtractPII<Type> = {
  [Property in keyof Type]: Type[Property] extends { pii: true } ? true : false;
};

type DBFields = {
  id: { format: "incrementing" };
  name: { type: string; pii: true };
};

type ObjectsNeedingGDPRDeletion = ExtractPII<DBFields>;
//   ^?
```
