# TypeScript Interfaces - Deep Dive (Part 1)

## Table of Contents
1. [Introduction to Interfaces](#1-introduction-to-interfaces)
2. [Creating Basic Interfaces](#2-creating-basic-interfaces)
3. [Types of Interfaces in TypeScript](#3-types-of-interfaces-in-typescript)
4. [Internal Working & Type System Mechanics](#4-internal-working--type-system-mechanics)
5. [Summary Checklist](#5-summary-checklist)

---

## 1. Introduction to Interfaces

In TypeScript, an **Interface** is a fundamental feature used to define the structure or contract that an object, class, or function must adhere to. It acts as a compile-time blueprint to enforce type safety across an application.

### Key Characteristics
* **Compile-time Only:** Interfaces exist purely in the TypeScript type-checking phase and produce **zero runtime code** (type erasure).
* **Structural Typing:** TypeScript uses duck typing ("if it walks like a duck and quacks like a duck, it's a duck"). An object satisfies an interface if it has all required properties, regardless of explicit inheritance.
* **Extensibility:** Interfaces can be extended by other interfaces or classes and support **declaration merging**.

---

## 2. Creating Basic Interfaces

### Basic Property Syntax
An interface is declared using the `interface` keyword:

```typescript
interface User {
  id: string;
  username: string;
  age: number;
}

const userA: User = {
  id: "usr_101",
  username: "alex_dev",
  age: 28,
};
```

### Optional & Readonly Properties
Interfaces support modifiers to declare properties as optional (`?`) or immutable (`readonly`):

```typescript
interface UserProfile {
  readonly id: string;           // Cannot be reassigned after creation
  username: string;
  email: string;
  avatarUrl?: string;            // Optional property (string | undefined)
}

const profile: UserProfile = {
  id: "usr_202",
  username: "sarah_k",
  email: "sarah@example.com",
};

// profile.id = "usr_999"; // ❌ Compiler Error: Cannot assign to 'id' because it is a read-only property.
profile.username = "sarah_m"; // ✅ Allowed
```

---

## 3. Types of Interfaces in TypeScript

TypeScript interfaces can model a variety of structures beyond standard object shapes.

```
       +-----------------------------------------------------------+
       |                 TYPES OF TS INTERFACES                    |
       +-----------------------------------------------------------+
       |                                                           |
       |  1. Object Interface        2. Function Interface         |
       |     { prop: type }             (arg: T): R                |
       |                                                           |
       |  3. Indexable Interface     4. Class Interface            |
       |     [key: string]: T           class Impl implements I    |
       |                                                           |
       |  5. Hybrid Interface                                      |
       |     Function + Properties                                 |
       +-----------------------------------------------------------+
```

### 3.1 Object Property Interfaces
The most common type, defining shape and types of properties on plain JavaScript objects.

```typescript
interface Address {
  street: string;
  city: string;
  zipCode: string;
}
```

### 3.2 Function / Callable Interfaces
Interfaces can define caller signatures for standalone functions.

```typescript
// Declares a function signature accepting two numbers and returning a boolean
interface Comparator {
  (a: number, b: number): boolean;
}

const isGreater: Comparator = (a, b) => a > b;
const isEqual: Comparator = (a, b) => a === b;
```

### 3.3 Indexable / Dictionary Interfaces
Used when object keys aren't known ahead of time, but values follow a specific type pattern.

```typescript
interface StringDictionary {
  [key: string]: string;
}

const environmentVars: StringDictionary = {
  NODE_ENV: "production",
  PORT: "8080",
  LOG_LEVEL: "debug",
};

interface ItemList {
  [index: number]: string; // Defines array-like numeric indexing
}

const tags: ItemList = ["typescript", "javascript", "backend"];
```

### 3.4 Class Contracts (`implements`)
Interfaces enforce public structural contracts on classes.

```typescript
interface Logger {
  log(message: string): void;
  error(message: string, code?: number): void;
}

class ConsoleLogger implements Logger {
  public log(message: string): void {
    console.log(`[LOG]: ${message}`);
  }

  public error(message: string, code?: number): void {
    console.error(`[ERR ${code ?? 500}]: ${message}`);
  }
}
```

### 3.5 Hybrid Interfaces
Defines objects that behave both as a function and as an object with additional properties (common in legacy JavaScript libraries like jQuery or Axios).

```typescript
interface Counter {
  (start: number): string; // Callable signature
  interval: number;         // Property signature
  reset(): void;            // Method signature
}

function getCounter(): Counter {
  const counter = ((start: number) => `Started at ${start}`) as Counter;
  counter.interval = 1000;
  counter.reset = () => console.log("Counter reset");
  return counter;
}

const myCounter = getCounter();
console.log(myCounter(10)); // "Started at 10"
myCounter.reset();          // "Counter reset"
```

---

## 4. Internal Working & Type System Mechanics

### 4.1 Structural Typing vs. Nominal Typing
Languages like Java/C# use **Nominal Typing** (type equivalence requires matching explicit declarations/names). TypeScript uses **Structural Typing** (type equivalence depends solely on shape).

```typescript
interface Point2D {
  x: number;
  y: number;
}

interface Vector2D {
  x: number;
  y: number;
}

function renderPoint(p: Point2D) {
  console.log(`Point at: ${p.x}, ${p.y}`);
}

const velocity: Vector2D = { x: 10, y: 20 };

// Valid in TypeScript because shapes match exactly, despite different interface names!
renderPoint(velocity);
```

### 4.2 Excessive Property Checks (Fresh Object Literals)
When assigning object literals directly, TypeScript applies strict **Excess Property Checks** to catch typos.

```typescript
interface BoxConfig {
  width: number;
  height: number;
}

function createBox(config: BoxConfig) {}

// ❌ Error: 'depth' does not exist in type 'BoxConfig'
// createBox({ width: 10, height: 20, depth: 30 });

// ✅ Bypass 1: Assigning to intermediate variable disables fresh literal check
const extraConfig = { width: 10, height: 20, depth: 30 };
createBox(extraConfig);

// ✅ Bypass 2: Use Index Signatures in interface definition
interface FlexibleBoxConfig {
  width: number;
  height: number;
  [key: string]: any;
}
```

### 4.3 Zero Transpilation Footprint (Type Erasure)

#### TypeScript Source
```typescript
interface PaymentRequest {
  amount: number;
  currency: string;
}

function processPayment(req: PaymentRequest): boolean {
  return req.amount > 0;
}
```

#### Transpiled JavaScript Output
```javascript
"use strict";
function processPayment(req) {
    return req.amount > 0;
}
```
Notice that the `interface PaymentRequest` completely disappears in JS.

---

## 5. Summary Checklist

* [ ] **Type Erasure:** Interfaces exist only during development/compilation and vanish in JavaScript outputs.
* [ ] **Structural Compatibility:** Object compatibility depends on property availability and matching types, not naming.
* [ ] **Modifiers:** Use `?` for optional properties and `readonly` for immutable fields.
* [ ] **Variants:** Use interfaces for objects, functions, indexable dictionaries, and class implementation contracts.