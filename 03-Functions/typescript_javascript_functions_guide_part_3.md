# 🧩 JavaScript & TypeScript Functions Guide: Part 3 - Arrow Functions & Advanced Lambdas

Arrow Functions (introduced in ES6) provide a concise syntax for writing function expressions and solve common scoping pitfalls by introducing **Lexical Scope Binding** for `this`, `arguments`, `super`, and `new.target`.

## 📑 Table of Contents

1. [Definition & Introduction](#1-definition--introduction)

2. [Declaration & Syntax Variations](#2-declaration--syntax-variations)

3. [Implicit Return Mechanics](#3-implicit-return-mechanics)

4. [Code Examples across Use Cases](#4-code-examples-across-use-cases)

5. [Lexical `this` vs Dynamic `this`](#5-lexical-this-vs-dynamic-this)

6. [Advanced Arrow Function Patterns & Mechanics](#6-advanced-arrow-function-patterns--mechanics)

7. [Limitations & When NOT to Use Arrow Functions](#7-limitations--when-not-to-use-arrow-functions)

---

## 1. Definition & Introduction

An **Arrow Function** (also known as a Lambda function) is an anonymous function expression defined using the fat arrow (`=>`) syntax.

### Key Architectural Differences

Unlike standard function declarations or expressions:

* **No Binding of `this`:** Inherits `this` from the lexically enclosing scope.

* **No `arguments` Object:** Uses surrounding scope `arguments` or ES6 Rest Parameters (`...args`).

* **Not Constructible:** Cannot be called with `new` and lacks a `.prototype` property.

* **No `super` or `new.target`:** Inherited from the lexical environment.

---

## 2. Declaration & Syntax Variations

Arrow functions adjust syntactically based on parameter count and function body complexity:

### A. Parameter Variations

```typescript
// 1. Single Parameter (Parentheses optional in plain JS, required in TS when typed)
const square = (x: number): number => x * x;

// 2. Multiple Parameters (Parentheses required)
const add = (a: number, b: number): number => a + b;

// 3. Zero Parameters (Empty parentheses required)
const getTimestamp = (): number => Date.now();

// 4. Rest Parameters & Destructuring
const sumAll = (...numbers: number[]): number => 
  numbers.reduce((acc, curr) => acc + curr, 0);

const printUser = ({ name, role }: { name: string; role: string }): string => 
  `${name} (${role})`;
```

### B. Body Variations

```typescript
// Concise Body (Implicit Return expression)
const multiply = (a: number, b: number): number => a * b;

// Block Body (Explicit 'return' statement required)
const processData = (data: number[]): number => {
  const filtered = data.filter((x) => x > 0);
  return filtered.reduce((a, b) => a + b, 0);
};
```

---

## 3. Implicit Return Mechanics

When an arrow function has a **concise body** (no curly braces `{}`), the expression following `=>` is automatically evaluated and returned.

### Returning Object Literals

To implicitly return an object literal, wrap the object in parentheses `({ ... })`. Otherwise, JavaScript parses the `{}` as a block body statement, causing runtime bugs or syntax errors.

```typescript
interface User {
  id: string;
  active: boolean;
}

// ❌ Incorrect: Treated as a function block with an unlabeled statement
// const makeUser = (id: string): User => { id: id, active: true }; 

// ✅ Correct: Wrapped in parentheses for implicit object return
const makeUser = (id: string): User => ({ id, active: true });

console.log(makeUser("USR_101")); // { id: 'USR_101', active: true }
```

---

## 4. Code Examples across Use Cases

### A. Functional Method Chaining

```typescript
const prices = [12.99, 5.50, 24.00, 3.25];

// Declarative pipeline using concise arrow functions
const formattedTotals = prices
  .filter((price) => price > 5.00)
  .map((price) => price * 1.1) // Add 10% tax
  .map((price) => `$${price.toFixed(2)}`);

console.log(formattedTotals); // [ '$14.29', '$26.40' ]
```

### B. Event Listeners & Asynchronous Callbacks

```typescript
class NotificationService {
  private serviceName = "EmailNotifier";

  public sendDelayedNotification(message: string, delayMs: number): void {
    // Arrow function captures 'this.serviceName' lexically from instance context
    setTimeout(() => {
      console.log(`[${this.serviceName}] Sent: ${message}`);
    }, delayMs);
  }
}

const notifier = new NotificationService();
notifier.sendDelayedNotification("Password Reset Request", 100);
```

---

## 5. Lexical `this` vs Dynamic `this`

Understanding lexical scope resolution is critical when selecting between standard functions and arrow functions.

```
       Standard Function (Dynamic 'this')             Arrow Function (Lexical 'this')
       ┌───────────────────────────────┐              ┌───────────────────────────────┐
       │ Bound at RUNTIME based on     │              │ Captured at DEFINITION time   │
       │ WHO invoked the function.     │              │ from enclosing scope.         │
       └───────────────┬───────────────┘              └───────────────┬───────────────┘
                       │                                              │
                       ▼                                              ▼
          obj.method() ──► 'this' = obj                  Inherits 'this' from
          fn()         ──► 'this' = global/undefined     parent lexical context
```

### Comparative Code Example

```typescript
const handler = {
  id: "EVENT_001",

  // 1. Standard Method: 'this' refers to 'handler'
  initStandard: function() {
    console.log("Standard Init ID:", this.id); // "EVENT_001"
    
    setTimeout(function() {
      // ❌ Dynamic 'this' points to global/Timeout object in standard callbacks
      // console.log("Standard Timeout ID:", this.id); // undefined
    }, 50);
  },

  // 2. Arrow Method in Callback: Preserves lexical 'this'
  initArrow: function() {
    setTimeout(() => {
      // ✅ 'this' is lexically resolved to 'handler'
      console.log("Arrow Timeout ID:", this.id); // "EVENT_001"
    }, 50);
  }
};

handler.initStandard();
handler.initArrow();
```

---

## 6. Advanced Arrow Function Patterns & Mechanics

### A. Currying with Arrow Functions

Currying transforms a function taking multiple arguments into a chain of single-argument functions. Arrow functions make curried pipelines compact.

```typescript
// Standard Curried Function
const multiplyStandard = function(a: number) {
  return function(b: number) {
    return a * b;
  };
};

// Concise Arrow Currying
const multiplyCurried = (a: number) => (b: number): number => a * b;

const double = multiplyCurried(2);
console.log(double(10)); // 20
console.log(multiplyCurried(3)(5)); // 15
```

### B. Async Arrow Functions

The `async` keyword precedes parameter list declarations for asynchronous arrow functions.

```typescript
const fetchJSON = async <T>(url: string): Promise<T> => {
  const response = await fetch(url);
  if (!response.ok) {
    throw new Error(`HTTP Error: ${response.status}`);
  }
  return (await response.json()) as T;
};
```

### C. Memory Footprint in Class Fields

Defining arrow functions as class fields attaches the function instance to **every created object instance** directly, rather than sharing it on the prototype chain.

```typescript
class Button {
  id = "btn-1";

  // ⚠️ Stored on EVERY instance (Higher memory allocation)
  handleClick = () => {
    console.log(`Clicked ${this.id}`);
  };

  // ✅ Stored ONCE on Prototype (Lower memory allocation)
  render(): void {
    console.log("Rendering button");
  }
}
```

---

## 7. Limitations & When NOT to Use Arrow Functions

| Anti-Pattern Scenario | Example Issue | Recommended Alternative |
| :--- | :--- | :--- |
| **Object Methods** | `{ id: 1, getId: () => this.id }` (`this` points to global/outer scope) | Use method syntax: `getId() { return this.id; }` |
| **Object Prototypes** | `User.prototype.getName = () => this.name;` | Standard function expression |
| **Dynamic Event Bindings** | `button.addEventListener('click', () => this.classList.toggle('active'))` | Standard function (accesses `this` element) |
| **Constructors** | `const Person = (name) => { this.name = name; }; new Person()` ❌ Throws Error | Standard function or `class` declaration |
| **Generator Functions** | Cannot use `yield` keyword inside arrow functions | Generator function syntax `function* ()` |
