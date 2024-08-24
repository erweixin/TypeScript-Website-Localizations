---
title: 通过现有类型创建新类型
layout: docs
permalink: /zh/docs/handbook/2/types-from-types.html
oneline: "关于如何从现有类型创建更多新类型的概述。"
---

TypeScript 的类型系统之所以非常强大，是因为它允许根据 _其他类型_ 来表达类型。

这个思想最简单的形式是泛型。此外，我们有各种可用的 _类型操作符_。
我们也可以根据我们已经拥有的 _值_ 来表达类型。

通过结合各种类型操作符，我们可以用简洁、可维护的方式表达复杂的操作和值。
在这个章节，我们将介绍如何根据现有类型或值表达新类型的方法。

- [范型](/zh/docs/handbook/2/generics.html) - 接收参数的类型
- [Keyof 类型操作符](/zh/docs/handbook/2/keyof-types.html) - 使用 `keyof` 操作符创建新类型
- [Typeof 类型操作符](/zh/docs/handbook/2/typeof-types.html) - 使用 `typeof` 操作符创建新类型
- [索引访问类型](/zh/docs/handbook/2/indexed-access-types.html) -使用 `Type['a']` 语法访问类型的子集
- [条件类型](/zh/docs/handbook/2/conditional-types.html) - 在类型系统中表现得像 `if` 语句的类型
- [映射类型](/zh/docs/handbook/2/mapped-types.html) - 通过映射现有类型的每个属性来创建类型
- [模板字面量类型](/zh/docs/handbook/2/template-literal-types.html) - 通过模板字面量字符串改变属性的映射类型
