---
title: keyof 类型操作符
layout: docs
permalink: /zh/docs/handbook/2/keyof-types.html
oneline: "在类型上下文中使用 keyof 操作符。"
---

## Keyof 类型操作符

`keyof` 操作符用于对象类型，它会生成该对象属性名的字符串或数字字面量联合类型。
例如，下面的类型 `P` 等同于 `type P = "x" | "y"`：

```ts twoslash
type Point = { x: number; y: number };
type P = keyof Point;
//   ^?
```

如果类型带有 `string` 或 `number` 索引签名，`keyof` 会返回这些类型：]

```ts twoslash
type Arrayish = { [n: number]: unknown };
type A = keyof Arrayish;
//   ^?

type Mapish = { [k: string]: boolean };
type M = keyof Mapish;
//   ^?
```

注意在这个例子中，`M` 的类型是 `string | number`。
这是因为 JavaScript 对象的键总是会被强制转换为字符串，所以 `obj[0]` 总是等同于 `obj["0"]`。

`keyof` 类型在与映射类型结合使用时特别有用，我们稍后会详细了解映射类型。
