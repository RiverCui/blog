---
title: '变量提升 vs 函数提升'
date: 2025-02-06
description: " "
---

先说下什么是变量提升，什么是函数提升。

## 变量提升
使用 **`var`** 声明的变量会被提升到当前作用域的顶部，只提升声明，不提升赋值。
`let` 和 `const` 声明的变量不会被提升，会产生暂时性死区（TDL）。

```js
// var 声明，变量提升
console.log(a)  // undefined
var a = 10

// let/cost 声明，变量不提升
console.log(b)  // Uncaught ReferenceError: b is not defined
let b = 100
```

## 函数提升
函数声明会被**完整**的提升到当前作用域顶部，包括**函数名**和**函数体**。
函数表达式不会被提升，因为本质上是变量赋值。

```js
// 函数提升
console.log(foo)  // ƒ foo() { console.log('Hello') }  // 打印结果是函数名和函数体
function foo() {
  console.log('Hello')
}

// var 声明的函数表达式，变量提升
console.log(foo)  // undefined  // 本质是变量提升，只提升声明，所以打印结果是 undefined
var foo = function() {
    console.log('hello')
}
```

## 函数提升优先级 > 变量提升优先级

实际上，是先进性函数提升，再进行变量提升的。我们可以测试一下：

```js
console.log(foo);  // ƒ foo() {}  // 输出函数体
function foo() {};
var foo;

console.log(foo);  // ƒ foo() {}  // 输出函数体
var foo;
function foo() {};
```

但是后面的函数声明会覆盖前面的函数声明，也同样测试下：
```js
console.log(foo)  // ƒ foo() { console.log('2') }  // 输出后面的函数声明
function foo() { console.log('1') }
function foo() { console.log('2') }
```

我们可以分析下载编译阶段时，JS引擎是如何处理函数声明和变量声明的。
```js
// 以这两个声明为例
console.log(foo);
var foo = 1;
function foo() {
  return "function";
}

// JS 引擎的处理步骤：
// 步骤1：首先提升函数声明
function foo() { return "function"; }

// 步骤2：然后处理变量声明（但因为已有同名函数声明，这步实际上被忽略）
// var foo; // 被忽略！

// 步骤3：执行阶段
console.log(foo); // 输出函数体
foo = 1; // 执行阶段：赋值覆盖之前的函数声明
```

注意，在最后的执行阶段，变量赋值会覆盖函数声明。

总结一下，要注意的有三点：
- 函数提升优先级高于变量提升
- 后面的函数声明会覆盖前面的函数声明
- 变量赋值会在执行阶段覆盖函数声明

函数提升、变量提升发生在**编译阶段**，变量赋值发生在**执行阶段**。理解了这句话就会很好理解上面三点。