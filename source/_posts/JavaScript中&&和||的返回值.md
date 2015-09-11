title: "JavaScript中'&&'和'||'的返回值"
date: 2015-07-19 12:17:00
tags: JavaScript
---
今天看见一个题目

```js
alert(1&&2)
```
初始觉得结果可能是`true`吧，任务逻辑运算符的返回值都是 `true` 或 `false` ，但结果是 `2`。

<!-- more -->

对 `2` 这个结果并不是很明白，翻了下 [ECMA spec](http://www.ecma-international.org/ecma-262/6.0/index.html#sec-binary-logical-operators-runtime-semantics-evaluation)
>
1. Let lref be the result of evaluating LogicalANDExpression.
1. Let lval be GetValue(lref).
1. Let lbool be ToBoolean(lval).
1. ReturnIfAbrupt(lbool).
1. If lbool is false, return lval.
1. Let rref be the result of evaluating BitwiseORExpression.
1. Return GetValue(rref).

第一个表达式值的bool值为false，则返回第一个表达式的值。否则返回第二个表达式的值。

因此可以发现下面几种结果很好理解了。

```js
alert(false && 2) // false
alert(0 && 2) // 0
alert(1 && function(){}) // function(){}
```
`||`同理