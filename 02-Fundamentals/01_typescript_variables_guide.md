# 📦 The Comprehensive Guide to Variables in TypeScript & JavaScript

Variables are named storage locations in memory used to hold data that can be retrieved, updated, or manipulated during program execution.

In TypeScript, variables build directly upon JavaScript's dynamic variable model while adding a compile-time static type system.

---

## 📑 Table of Contents
1. [Definition & Core Concepts](#1-definition--core-concepts)
2. [Standard Naming Rules & Conventions](#2-standard-naming-rules--conventions)
3. [Variable Types: Primitive vs. Reference](#3-variable-types-primitive-vs-reference)
4. [Internal Working & Memory Allocation](#4-internal-working--memory-allocation)
5. [Syntax & Declarations](#5-syntax--declaration)
6. [Scope, Redeclaration & Reinitialization](#6-scope-redeclaration--reinitialization)
7. [The Definitive Comparison: `var` vs. `let` vs. `const`](#7-the-definitive-comparison-var-vs-let-vs-const)
8. [Hoisting & The Temporal Dead Zone (TDZ)](#8-hoisting--the-temporal-dead-zone-tdz)

---

## 1. Definition & Core Concepts

A variable acts as a pointer or identifier to a specific memory address storing a value. 

```
Variable Identifier (Name) ──► [ Memory Address (e.g. 0x002F) ] ──► Value Stored (e.g. 42)
```

In TypeScript, a variable declaration consists of four main parts:
1. **Declaration Keyword:** (`var`, `let`, or `const`)
2. **Identifier:** Name used to refer to the variable.
3. **Type Annotation (Optional in TS):** Restricts the kind of data the variable can hold.
4. **Initializer (Optional):** Assigns an initial value.

---

## 2. Standard Naming Rules & Conventions

### 📏 Mandatory Syntax Rules
* Must begin with a letter (`a-z`, `A-Z`), underscore (`_`), or dollar sign (`$`).
* Cannot start with a digit (`0-9`).
* Case-sensitive (`myVar` and `myvar` are distinct).
* Cannot use reserved ECMAScript / TypeScript keywords (`class`, `function`, `return`, `interface`, etc.).

### 🎨 Industry Conventions
* **camelCase:** Standard for variables and function names (`userId`, `totalPrice`).
* **PascalCase:** Reserved for classes, interfaces, types, enums, and components (`UserProfile`, `PaymentStatus`).
* **UPPER_SNAKE_CASE:** Reserved for compile-time or global constants (`MAX_ATTEMPTS`, `API_BASE_URL`).

---

## 3. Variable Types: Primitive vs. Reference

JavaScript and TypeScript handle data types differently in terms of storage and assignment:

```
                          Data Types
                              │
         ┌────────────────────┴────────────────────┐
         ▼                                         ▼
  Primitive Types                           Reference Types
  (Stored by Value on Stack)                (Stored by Reference on Heap)
  • number, string, boolean                 • Objects, Arrays
  • bigint, symbol, null, undefined         • Functions, Maps, Sets
```

### A. Primitive Types (Value Types)
* Immutable values stored directly in **Stack Memory**.
* When copied, a **new independent copy** of the value is created.

```typescript
let x = 10;
let y = x; // Copy by value
y = 20;

console.log(x); // 10 (x is unchanged)
console.log(y); // 20
```

### B. Reference Types (Complex Types)
* Mutable collections stored in **Heap Memory**.
* The variable on the Stack stores a pointer (reference address) pointing to the heap memory location.
* Copying copies the **pointer**, not the actual object.

```typescript
let obj1 = { name: "Alice" };
let obj2 = obj1; // Copy by reference

obj2.name = "Bob";

console.log(obj1.name); // "Bob" (Both variables point to the same object!)
```

---

## 4. Internal Working & Memory Allocation

During JavaScript/TypeScript execution (managed by engines like V8), memory is divided into two main structures: **Call Stack** and **Memory Heap**.

```
             CALL STACK                              MEMORY HEAP
┌─────────────────────────────────┐           ┌───────────────────────┐
│ Frame: main()                   │           │                       │
│ ┌─────────────────────────────┐ │           │  {                    │
│ │ user: 0x004A2 (Pointer) ────┼─┼───────────┼─►  name: "Alex",     │
│ ├─────────────────────────────┤ │           │    age: 28            │
│ │ count: 42 (Value)           │ │           │  }                    │
│ └─────────────────────────────┘ │           │                       │
└─────────────────────────────────┘           └───────────────────────┘
```

### Stack Memory
* **Structure:** Last-In, First-Out (LIFO).
* **Speed:** Extremely fast access.
* **Storage:** Stores static data, function call frames, primitive values, and pointers/references to complex objects.

### Heap Memory
* **Structure:** Unstructured, dynamic memory pool.
* **Speed:** Slower allocation and access than stack.
* **Storage:** Stores complex, dynamically sized objects, arrays, and functions.
* **Garbage Collection (GC):** Periodically reclaims heap memory when objects no longer have active references pointing to them from the stack.

---

## 5. Syntax & Declaration

TypeScript allows explicit type annotations or implicit type inference.

### Explicit Type Annotations
```typescript
let age: number = 25;
let username: string = "Sarah";
let isVerified: boolean = true;
```

### Type Inference
If initialized upon declaration, TypeScript automatically infers the variable's type:
```typescript
let count = 100; // Inferred as 'number'
// count = "hello"; // ❌ Type Error: Type 'string' is not assignable to type 'number'.
```

---

## 6. Scope, Redeclaration & Reinitialization

### Scope Definitions
1. **Global Scope:** Accessible anywhere in the application.
2. **Function Scope:** Accessible only within the body of the declaring function (`var`).
3. **Block Scope:** Accessible only within the enclosing `{ ... }` block (`let` and `const`).

---

## 7. The Definitive Comparison: `var` vs. `let` vs. `const`

| Property | `var` | `let` | `const` |
| :--- | :--- | :--- | :--- |
| **Scope** | Function Scope | Block Scope | Block Scope |
| **Hoisting Behavior** | Hoisted (initialized as `undefined`) | Hoisted (Uninitialized in TDZ) | Hoisted (Uninitialized in TDZ) |
| **Redeclaration in same scope** | ✅ Allowed | ❌ Syntax Error | ❌ Syntax Error |
| **Reinitialization (Reassignment)** | ✅ Allowed | ✅ Allowed | ❌ TypeError |
| **Must initialize on declaration** | ❌ Optional | ❌ Optional | ✅ Required |

### Code Examples

```typescript
// --- 1. Redeclaration ---
var a = 1;
var a = 2; // ✅ Allowed with var

let b = 1;
// let b = 2; // ❌ Error: Cannot redeclare block-scoped variable 'b'.

// --- 2. Reinitialization ---
let score = 10;
score = 20; // ✅ Allowed with let

const maxScore = 100;
// maxScore = 200; // ❌ Error: Cannot assign to 'maxScore' because it is a constant.

// --- 3. Mutation vs. Reassignment (const) ---
const person = { name: "John" };
person.name = "Jane"; // ✅ Allowed! (Mutating heap object property)
// person = { name: "Jane" }; // ❌ Error! (Reassigning pointer variable)
```

---

## 8. Hoisting & The Temporal Dead Zone (TDZ)

### What is Hoisting?
**Hoisting** is JavaScript's default behavior of moving variable and function declarations to the top of their containing scope during the compilation phase, before execution starts.

> **Crucial Rule:** Only the **declaration** is hoisted, NOT the **initialization**.

### A. `var` Hoisting Behavior
When `var` is hoisted, it is automatically initialized with `undefined`.

```javascript
console.log(myVar); // Output: undefined (No error thrown!)
var myVar = 10;

// Under the hood, the engine interprets this as:
// var myVar;           <-- Hoisted to top, initialized as undefined
// console.log(myVar);
// myVar = 10;
```

### B. `let` and `const` Hoisting + Temporal Dead Zone (TDZ)
`let` and `const` are also hoisted, but they are **NOT** initialized with `undefined`. They remain in an uninitialized state inside the **Temporal Dead Zone (TDZ)** from the start of the block until execution reaches their declaration line.

```
┌──────────────────────────────────────────┐
│ Block Scope Start                        │
│                                          │
│  ├─── Temporal Dead Zone (TDZ) ────┤     │
│  │ Accessing variable here throws  │     │
│  │ ReferenceError!                 │     │
│  └─────────────────────────────────┘     │
│                                          │
│  let myLet = 42;  <-- Declaration Line   │
│                                          │
│  Access allowed after this point ✅     │
└──────────────────────────────────────────┘
```

#### TDZ Example:
```typescript
function testTDZ() {
  // TDZ Start for 'item'
  console.log(item); // ❌ ReferenceError: Cannot access 'item' before initialization
  
  let item = "Laptop"; // TDZ End for 'item'
  console.log(item); // ✅ "Laptop"
}
```

---

## 💡 Best Practices Summary

1. **Avoid `var`:** Always prefer `const` by default, and use `let` when you explicitly plan to reassign a value.
2. **Use Type Annotations:** Leverage TypeScript types (`let count: number`) whenever the type cannot be clearly inferred automatically.
3. **Keep `const` Immutable:** Use `Object.freeze()` or TypeScript `readonly` types if you need full immutability for object properties.