---
title: 模板字面量类型
layout: docs
permalink: /zh/docs/handbook/2/template-literal-types.html
oneline: "通过模板字面量字符串生成改变属性的映射类型。"
---

模板字面量类型基于[字符串字面量类型]((/zh/docs/handbook/2/everyday-types.html#literal-types))，并具有通过联合类型扩展到多个字符串的能力。

它们有着与 [JavaScript中的模板字面量字符串]((https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals)) 相同的语法，但用于类型位置。
在具体字面量类型中使用模板字面量时，通过拼接内容可以产生新的字符串字面量类型。

```ts twoslash
type World = "world";

type Greeting = `hello ${World}`;
//   ^?
```

当在插入的位置使用联合类型时，类型是每个联合成员可以表示的所有可能字符串字面量的集合：

```ts twoslash
type EmailLocaleIDs = "welcome_email" | "email_heading";
type FooterLocaleIDs = "footer_title" | "footer_sendoff";

type AllLocaleIDs = `${EmailLocaleIDs | FooterLocaleIDs}_id`;
//   ^?
```

对于模板字面量中的每个插值位置，联合类型将相互乘积：

```ts twoslash
type EmailLocaleIDs = "welcome_email" | "email_heading";
type FooterLocaleIDs = "footer_title" | "footer_sendoff";
// ---cut---
type AllLocaleIDs = `${EmailLocaleIDs | FooterLocaleIDs}_id`;
type Lang = "en" | "ja" | "pt";

type LocaleMessageIDs = `${Lang}_${AllLocaleIDs}`;
//   ^?
```

我们通常建议人们在处理大型字符串联合时提前生成，但在较小的情况下这仍然很有用。

### 类型中的字符串联合

模板字面量的威力在于根据类型内部的信息定义新的字符串。

考虑一个情况，其中一个函数(`makeWatchedObject`)向传入的对象添加了一个新的函数
称为 `on()`。在 JavaScript 中，它的调用可能看起来像这样：`makeWatchedObject(baseObject)`。我们可以想象 `baseObject` 看起来像这样：

```ts twoslash
// @noErrors
const passedObject = {
  firstName: "Saoirse",
  lastName: "Ronan",
  age: 26,
};
```

将被添加到基对象上的 `on` 函数需要两个参数，一个 `eventName` （一个 `string`）和一个 `callback`（一个 `function`）。

`eventName` 应该是 `attributeInThePassedObject + "Changed"` 的形式；因此，`firstNameChanged` 是由 `baseObject` 的 `firstName` 属性派生出来的。

当被调用时，`callback` 函数：

  * 应该被传入一个与名称 `attributeInThePassedObject` 关联的类型的值；因此，由于 `firstName` 被键入为 `string`，因此 `firstNameChanged` 事件的回调期望在调用时传入一个 `string`。同样，与 `age` 相关的事件应该期望被调用时传入一个 `number` 参数。
  * 应该有 `void` 返回类型（为了演示的简单性）

`on()` 的简单函数签名可能是：`on(eventName: string, callback: (newValue: any) => void)`。然而，在前面的描述中，我们确定了我们希望在代码中记录的重要类型约束。模板字面量类型让我们可以在代码中引入这些约束。

```ts twoslash
// @noErrors
declare function makeWatchedObject(obj: any): any;
// ---cut---
const person = makeWatchedObject({
  firstName: "Saoirse",
  lastName: "Ronan",
  age: 26,
});

// makeWatchedObject has added `on` to the anonymous Object

person.on("firstNameChanged", (newValue) => {
  console.log(`firstName was changed to ${newValue}!`);
});
```

注意，`on` 监听的事件是 `"firstNameChanged"`，而不只是 `"firstName"`。如果我们能确保合格的事件名集合受到被观察对象中属性名联合与 "Changed" 相加的约束，那么我们对 `on()` 的粗略说明就可以变得更加健壮。虽然我们对在JavaScript中进行这样的计算是方便的，例如 ``Object.keys(passedObject).map(x => `${x}Changed`)``，但是模板字面量 _在类型系统内部_ 提供了类似于字符串操作的方法：

```ts twoslash
type PropEventSource<Type> = {
    on(eventName: `${string & keyof Type}Changed`, callback: (newValue: any) => void): void;
};

/// Create a "watched object" with an `on` method
/// so that you can watch for changes to properties.
declare function makeWatchedObject<Type>(obj: Type): Type & PropEventSource<Type>;
```

有了这个，我们可以构建一些在使用错误属性时有错误提示的逻辑：

```ts twoslash
// @errors: 2345
type PropEventSource<Type> = {
    on(eventName: `${string & keyof Type}Changed`, callback: (newValue: any) => void): void;
};

declare function makeWatchedObject<T>(obj: T): T & PropEventSource<T>;
// ---cut---
const person = makeWatchedObject({
  firstName: "Saoirse",
  lastName: "Ronan",
  age: 26
});

person.on("firstNameChanged", () => {});

// Prevent easy human error (using the key instead of the event name)
person.on("firstName", () => {});

// It's typo-resistant
person.on("frstNameChanged", () => {});
```

### 使用模板字面量进行推断

注意，我们并未从原始传入对象中提供的全部信息中获益。考虑 `firstName` 的改变（即 `firstNameChanged` 事件），我们期望回调将接收一个 `string` 类型的参数。同样，对 `age` 的改变的回调应该接收一个 `number` 类型的参数。我们在为 `callback` 的参数简单地使用了 `any` 进行类型注解。同样，模板字面量类型让我们能确保属性的数据类型将与该属性的回调的第一个参数的类型相同。

实现这一点的关键洞察力是：我们可以使用带有范型的函数，使得：

1. 第一个参数中使用的字面量被捕获为一个字面类型
2. 那个字面类型可以作为范型的有效属性的联合类型进行验证
3. 使用索引访问，可以在范型的结构中查找验证属性的类型
4. 这个类型信息 _然后_ 可以应用于确保回调函数的参数类型与该类型相同

```ts twoslash
type PropEventSource<Type> = {
    on<Key extends string & keyof Type>
        (eventName: `${Key}Changed`, callback: (newValue: Type[Key]) => void): void;
};

declare function makeWatchedObject<Type>(obj: Type): Type & PropEventSource<Type>;

const person = makeWatchedObject({
  firstName: "Saoirse",
  lastName: "Ronan",
  age: 26
});

person.on("firstNameChanged", newName => {
    //                        ^?
    console.log(`new name is ${newName.toUpperCase()}`);
});

person.on("ageChanged", newAge => {
    //                  ^?
    if (newAge < 0) {
        console.warn("warning! negative age");
    }
})
```

这里我们使 on 成为了一个范型方法。Here we made `on` into a generic method.

当用户用字符串 · 调用时，TypeScript 会尝试推断 Key 的正确类型。
为此，它会将 `Key` 的内容与 `"Changed"` 之前的内容进行匹配，并推断出字符串 `"firstName"`。
一旦 TypeScript 弄清楚这一点，`on` 方法就可以获取原始对象上 `firstName` 的类型，这种情况下是 `string`。
同样，当用 `"ageChanged"` 调用时，TypeScript 找到 `age` 属性的类型，这是 `number`。

推断可以以不同的方式组合，通常是为了解构字符串，并以不同的方式重新构建它们。

## 内置的字符串处理类型

为了帮助字符串处理，TypeScript 包括了一套可以用于字符串处理的类型。这些类型内置到编译器中以提高性能，并且不会在与TypeScript 一起提供的 `.d.ts` 文件中找到。

### `Uppercase<StringType>`

将字符串中的每个字符转换为小写版本。

##### Example

```ts twoslash
type Greeting = "Hello, world"
type ShoutyGreeting = Uppercase<Greeting>
//   ^?

type ASCIICacheKey<Str extends string> = `ID-${Uppercase<Str>}`
type MainID = ASCIICacheKey<"my_app">
//   ^?
```

### `Lowercase<StringType>`

将字符串中的每个字符转换为小写版本。

##### Example

```ts twoslash
type Greeting = "Hello, world"
type QuietGreeting = Lowercase<Greeting>
//   ^?

type ASCIICacheKey<Str extends string> = `id-${Lowercase<Str>}`
type MainID = ASCIICacheKey<"MY_APP">
//   ^?
```

### `Capitalize<StringType>`

将字符串中的首字符转换为大写版本。

##### Example

```ts twoslash
type LowercaseGreeting = "hello, world";
type Greeting = Capitalize<LowercaseGreeting>;
//   ^?
```

### `Uncapitalize<StringType>`

将字符串中的首字符转换为小写版本。

##### Example

```ts twoslash
type UppercaseGreeting = "HELLO WORLD";
type UncomfortableGreeting = Uncapitalize<UppercaseGreeting>;
//   ^?
```

<details>
    <summary>内置字符串处理类型的技术细节</summary>
    <p>截止到 TypeScript 4.1，这些内置函数的代码直接使用 JavaScript 字符串运行时函数进行操作，并且不考虑本地化设置。</p>
    <code><pre>
function applyStringMapping(symbol: Symbol, str: string) {
    switch (intrinsicTypeKinds.get(symbol.escapedName as string)) {
        case IntrinsicTypeKind.Uppercase: return str.toUpperCase();
        case IntrinsicTypeKind.Lowercase: return str.toLowerCase();
        case IntrinsicTypeKind.Capitalize: return str.charAt(0).toUpperCase() + str.slice(1);
        case IntrinsicTypeKind.Uncapitalize: return str.charAt(0).toLowerCase() + str.slice(1);
    }
    return str;
}</pre></code>
</details>
