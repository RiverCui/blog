---
title: 'Understanding Hoisting in JavaScript: Variables and Functions'
date: 2025-02-06
description: " "
---

> Note: This article was translated from Chinese to English by Claude AI (Anthropic).

## Variable Hoisting
Variables declared with **`var`** are hoisted to the top of their current scope. Only the declaration is hoisted, not the initialization.
Variables declared with `let` and `const` are not hoisted and create a Temporal Dead Zone (TDZ).

```js
// Variable hoisting with var
console.log(a)  // undefined
var a = 10

// No hoisting with let/const
console.log(b)  // Uncaught ReferenceError: b is not defined
let b = 100
```

## Function Hoisting
Function declarations are **completely** hoisted to the top of their current scope, including both the **function name** and **function body**.
Function expressions are not hoisted since they are essentially variable assignments.

```js
// Function hoisting
console.log(foo)  // ƒ foo() { console.log('Hello') }  // Prints function name and body
function foo() {
  console.log('Hello')
}

// Function expression with var - only variable hoisting occurs
console.log(foo)  // undefined  // Only declaration is hoisted, so undefined is printed
var foo = function() {
    console.log('hello')
}
```

## Function Hoisting Takes Precedence Over Variable Hoisting

JavaScript engine performs function hoisting before variable hoisting. Let's examine this behavior:

```js
console.log(foo);  // ƒ foo() {}  // Outputs function body
function foo() {};
var foo;

console.log(foo);  // ƒ foo() {}  // Outputs function body
var foo;
function foo() {};
```

However, subsequent function declarations override previous ones:
```js
console.log(foo)  // ƒ foo() { console.log('2') }  // Outputs the latter function declaration
function foo() { console.log('1') }
function foo() { console.log('2') }
```

Let's analyze how the JavaScript engine processes function and variable declarations during the compilation phase:
```js
// Example declarations
console.log(foo);
var foo = 1;
function foo() {
  return "function";
}

// JavaScript engine processing steps:
// Step 1: Function declarations are hoisted first
function foo() { return "function"; }

// Step 2: Variable declarations are processed (but ignored if same name exists)
// var foo; // ignored!

// Step 3: Execution phase
console.log(foo); // outputs function body
foo = 1; // execution phase: assignment overwrites previous function declaration
```

Note that during the final execution phase, variable assignments will override function declarations.

Key points to remember:
- Function hoisting has higher priority than variable hoisting
- Later function declarations override earlier ones
- Variable assignments during execution override function declarations

Function and variable hoisting occur during the **compilation phase**, while variable assignment happens during the **execution phase**. Understanding this distinction helps clarify the above three points.