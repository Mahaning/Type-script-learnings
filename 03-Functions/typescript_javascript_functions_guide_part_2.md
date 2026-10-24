# 🧩 JavaScript & TypeScript Functions Guide: Part 2 - Anonymous Functions & Advanced Patterns

Anonymous functions—functions without a declared identifier name—are fundamental to functional programming, event handling, asynchronous callbacks, and scope isolation in JavaScript and TypeScript.

## 📑 Table of Contents

1. [Definition & Introduction](#1-definition--introduction)

2. [Types & Variations of Anonymous Functions](#2-types--variations-of-anonymous-functions)

3. [Standard Anonymous Function Expressions](#3-standard-anonymous-function-expressions)

4. [Arrow Functions (ES6 Lambda Expressions)](#4-arrow-functions-es6-lambda-expressions)

5. [Immediately Invoked Function Expressions (IIFE)](#5-immediately-invoked-function-expressions-iife)

6. [Higher-Order Anonymous Functions & Callbacks](#6-higher-order-anonymous-functions--callbacks)

7. [Advanced Anonymous Functions & Engine Mechanics](#7-advanced-anonymous-functions--engine-mechanics)

8. [Comparison Matrix: Anonymous vs. Named Functions](#8-comparison-matrix-anonymous-vs-named-functions)

---

## 1. Definition & Introduction

An **Anonymous Function** is a function created without a binding name identifier at its declaration site. 

Unlike Named Functions (Function Declarations), anonymous functions are evaluated at runtime as **Function Expressions**. Because they lack a top-level name identifier in the current scope, they are typically:

* Assigned to a variable or constant (`const fn = function() {}`).

* Passed directly as arguments to higher-order functions (callbacks).

* Returned from other functions (factories / closures).

* Invoked immediately upon creation (IIFE).

---

## 2. Types & Variations of Anonymous Functions

In JavaScript/TypeScript, anonymous functions manifest in four major syntax variations:

```
                          ┌─────────────────────────────────────────────────┐
                          │         Anonymous Function Paradigms            │
                          └────────────────────────┬────────────────────────┘
                                                   │
         ┌──────────────────────┬──────────────────┴───┬──────────────────────┐
         ▼                      ▼                      ▼                      ▼
┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐
│ standard Function│  │ Arrow Function   │  │ Immediately      │  │ Method           │
│ Expression       │  │ Expression       │  │ Invoked (IIFE)   │  │ Shorthand        │
│                  │  │                  │  │                  │  │                  │
│ const f =        │  │ const f =        │  │ (function() {    │  │ const obj = {    │
│  function() {}   │  │  () => {}        │  │  // ...          │  │  run() {}        │
│                  │  │                  │  │ })();            │  │ }                │
└──────────────────┘  └──────────────────┘  └──────────────────┘  └──────────────────┘
```

---

## 3. Standard Anonymous Function Expressions

### Definition & Syntax

A standard anonymous function expression uses the `function` keyword, specifies parameters and body, but omits the identifier name.

```typescript
// Syntax:
const variableName = function(param1: Type1, param2: Type2): ReturnType {
  // Body execution
  return value;
};
```

### Key Characteristic: Hoisting Behavior

Unlike Named Function Declarations, function expressions are **not hoisted** with their body. The variable declaration is hoisted (as `undefined` for `var`, or uninitialized temporal dead zone for `let`/`const`), but the function assignment occurs only when execution reaches that line.

```typescript
// ❌ Error: Cannot access 'calculateDiscount' before initialization
// calculateDiscount(100);

const calculateDiscount = function(price: number): number {
  return price * 0.85;
};

// ✅ Works fine after declaration
console.log(calculateDiscount(100)); // 85
```

### Example with Parameters & TS Types

```typescript
const formatCurrency = function(amount: number, currencySymbol: string = "$"): string {
  return `${currencySymbol}${amount.toFixed(2)}`;
};

console.log(formatCurrency(49.9)); // "$49.90"
```

---

## 4. Arrow Functions (ES6 Lambda Expressions)

### Definition & Syntax

Introduced in ES6, **Arrow Functions** provide a compact syntax for anonymous functions and introduce **Lexical `this` Binding**.

```typescript
// Explicit return syntax:
const multiply = (a: number, b: number): number => {
  return a * b;
};

// Implicit return shorthand (single expression):
const double = (x: number): number => x * 2;

// Implicit return of Object Literals (wrapped in parentheses):
const createUser = (id: string, role: string) => ({ id, role });
```

### Lexical `this` vs Dynamic `this`

Standard anonymous functions bind `this` dynamically depending on *how* they are called. Arrow functions do **not** have their own `this` context; they inherit `this` lexically from their enclosing parent scope.

```typescript
class Timer {
  seconds = 0;

  start() {
    // Standard anonymous function loses 'this' binding inside setInterval
    // setInterval(function() { this.seconds++; }, 1000); // ❌ 'this' is undefined or global

    // Arrow function captures 'this' lexically from Timer instance
    setInterval(() => {
      this.seconds++;
      console.log(`Elapsed: ${this.seconds}s`);
    }, 1000);
  }
}
```

---

## 5. Immediately Invoked Function Expressions (IIFE)

### Definition

An **IIFE** (pronounced "iffy") is an anonymous function executed immediately after its definition.

### Primary Use Case

Creating an isolated, private execution scope to avoid polluting global scope variables and prevent variable collision in legacy or module-less scripts.

### Syntax Variations

```typescript
// 1. Standard IIFE Syntax
(function(version: string) {
  const privateToken = "SECRET_12345";
  console.log(`App Initialized v${version} [Token Active]`);
})("1.0.0");

// 2. Arrow Function IIFE Syntax
((status: boolean) => {
  const mode = status ? "Production" : "Development";
  console.log(`System running in ${mode} mode.`);
})(true);

// 3. IIFE with Return Value
const AppState = (() => {
  let counter = 0; // Private variable via closure
  return {
    increment: () => ++counter,
    getVal: () => counter
  };
})();

console.log(AppState.increment()); // 1
console.log(AppState.getVal());    // 1
```

---

## 6. Higher-Order Anonymous Functions & Callbacks

Anonymous functions are heavily used as **Callbacks** passed into Higher-Order Functions (HOFs).

### A. Array Processing Iterators

```typescript
interface Transaction {
  id: string;
  amount: number;
  type: "credit" | "debit";
}

const ledger: Transaction[] = [
  { id: "TX1", amount: 150, type: "credit" },
  { id: "TX2", amount: 50, type: "debit" },
  { id: "TX3", amount: 200, type: "credit" }
];

// Passing inline anonymous arrow callbacks
const netCreditTotal = ledger
  .filter((tx) => tx.type === "credit")
  .map((tx) => tx.amount)
  .reduce((acc, amount) => acc + amount, 0);

console.log(`Total Credits: $${netCreditTotal}`); // "Total Credits: $350"
```

### B. Custom Higher-Order Factory Function

```typescript
// Function that accepts an anonymous logger callback
function executeOperation(
  a: number, 
  b: number, 
  operation: (x: number, y: number) => number,
  onComplete: (result: number) => void
): void {
  const res = operation(a, b);
  onComplete(res);
}

// Invocation using inline anonymous functions
executeOperation(
  12, 
  4, 
  (x, y) => x / y,                      // Anonymous operation callback
  (result) => console.log(`Result: ${result}`) // Anonymous logger callback
);
```

---

## 7. Advanced Anonymous Functions & Engine Mechanics

### A. Dynamic Name Inference in V8

Although anonymous functions are defined without a explicit name identifier, modern JS engines (V8, SpiderMonkey) perform **Inferred Name Tracking** (`.name` property) for stack trace debugging.

```typescript
const myUnNamedFn = function() {};
console.log(myUnNamedFn.name); // Output: "myUnNamedFn" (Inferred from variable key)

const obj = {
  action: () => {}
};
console.log(obj.action.name); // Output: "action"

// True fully anonymous function (no inferred variable assignment)
console.log((function() {}).name); // Output: "" (empty string)
```

### B. Memory Footprint & Garbage Collection in Loops

Creating anonymous functions inside tight or hot loops generates a new function object reference on the Heap *per iteration*. This causes unnecessary allocation overhead and increased Garbage Collection (GC) pressure.

```typescript
// ❌ Poor Performance Pattern: Re-allocating function per iteration
function badLoopProcess(data: number[]) {
  for (let i = 0; i < data.length; i++) {
    // Creates N anonymous function closures in Heap
    setTimeout(() => {
      console.log(`Item: ${data[i]}`);
    }, 100);
  }
}

// ✅ Optimized Pattern: Reuse named or outer-bound callback function
const logItem = (item: number) => console.log(`Item: ${item}`);

function optimizedLoopProcess(data: number[]) {
  for (let i = 0; i < data.length; i++) {
    setTimeout(logItem, 100, data[i]);
  }
}
```

### C. Constructor Capability & Prototype Differences

| Feature | Standard Anonymous Function (`function() {}`) | Arrow Function (`() => {}`) |
| :--- | :--- | :--- |
| **`prototype` Property** | Exists (`{ constructor: ... }`) | `undefined` |
| **Constructor Usage (`new`)** | ✅ Allowed | ❌ Throws `TypeError: ... is not a constructor` |
| **`arguments` Object** | ✅ Available locally | ❌ Not available (use Rest `...args`) |
| **`this` Binding** | Dynamic (Caller context) | Lexical (Outer scope) |
| **`super` Binding** | No | Lexical access in class methods |

---

## 8. Comparison Matrix: Anonymous vs. Named Functions

| Criterion | Named Function (`function foo() {}`) | Anonymous Expression (`const foo = function() {}`) | Arrow Function (`const foo = () => {}`) |
| :--- | :--- | :--- | :--- |
| **Hoisting** | Fully Hoisted (Can call before line) | Variable Hoisted only (`undefined` / TDZ) | Variable Hoisted only (`undefined` / TDZ) |
| **Name Property** | Explicitly set to identifier | Inferred from variable or empty | Inferred from variable or empty |
| **`this` Context** | Dynamic (Depends on caller) | Dynamic (Depends on caller) | Lexical (Inherited from parent scope) |
| **Constructible (`new`)**| Yes | Yes | No |
| **Use Case Primary** | Main module API methods, utility functions | Callbacks, assigned strategies | Event handlers, inline transformations, array methods |
| **Readability** | High for top-level code | Medium | High for short operational pipelines |