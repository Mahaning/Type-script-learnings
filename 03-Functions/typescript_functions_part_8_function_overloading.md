# TypeScript Functions (Part 8): Function Overloading

Welcome to **Part 8** of our TypeScript Functions series. In this guide, we will explore **Function Overloading**—a powerful TypeScript feature that allows a single function to be called in different ways with varying parameter types, parameter counts, or return types, while maintaining strict compile-time type safety.

---

## Table of Contents

1. [Definition & Introduction](#1-definition--introduction)
2. [Anatomy of Overload Signatures vs. Implementation Signature](#2-anatomy-of-overload-signatures-vs-implementation-signature)
3. [Internal Working & Compilation Mechanics](#3-internal-working--compilation-mechanics)
4. [Overloading Types & Basic Examples](#4-overloading-types--basic-examples)
5. [Overloading with Optional & Default Parameters](#5-overloading-with-optional--default-parameters)
6. [Complex Real-World Scenarios](#6-complex-real-world-scenarios)
7. [Summary & Best Practices](#7-summary--best-practices)

---

## 1. Definition & Introduction

### What is Function Overloading?

In statically typed languages like C++ or Java, **function overloading** means defining multiple functions with the same name but different signatures, and the runtime executes the appropriate function based on the arguments passed.

In TypeScript, JavaScript's dynamic nature means there is **only ever one single underlying implementation function at runtime**. However, TypeScript provides **Overload Signatures** at compile time so the TypeScript compiler can enforce type checking for different valid calling conventions.

```
                  +---------------------------------------------------+
                  |           Callers / Invocation Code               |
                  +---------------------------------------------------+
                       |                                   |
    Call Version A:    | (string)                          | Call Version B: (number, number)
                       v                                   v
                  +---------------------------------------------------+
                  |            Compile-Time Overload 1                |
                  |            Compile-Time Overload 2                |
                  +---------------------------------------------------+
                                           |
                                           v
                  +---------------------------------------------------+
                  |            Single Shared Implementation           |
                  |           (Handles all cases at runtime)          |
                  +---------------------------------------------------+
```

---

## 2. Anatomy of Overload Signatures vs. Implementation Signature

Function overloading in TypeScript consists of two distinct parts:

1. **Overload Signatures**: One or more function headers without a body. These define the public API available to consumers.
2. **Implementation Signature**: The actual function header with a body. It must be compatible with **all** overload signatures, but it is **hidden** from consumers.

```typescript
// 1. OVERLOAD SIGNATURES (Public API)
function formatInput(value: string): string;
function formatInput(value: number): string;

// 2. IMPLEMENTATION SIGNATURE (Private/Internal mechanics)
function formatInput(value: string | number): string {
  if (typeof value === "string") {
    return value.trim().toUpperCase();
  }
  return value.toFixed(2);
}

// ✅ VALID CALLS
const a = formatInput("  hello "); // Evaluated against Overload 1
const b = formatInput(42);         // Evaluated against Overload 2

// ❌ INVALID CALL
// formatInput(true); 
// Error: No overload matches this call.
```

---

## 3. Internal Working & Compilation Mechanics

### How TypeScript Compiles Overloads

When compiled to JavaScript, all overload signatures are stripped completely. Only the implementation function remains in the output JavaScript file.

#### TypeScript Code:
```typescript
function multiply(x: number, y: number): number;
function multiply(x: string, y: number): string;
function multiply(x: number | string, y: number): number | string {
  if (typeof x === "number") {
    return x * y;
  }
  return x.repeat(y);
}
```

#### Compiled JavaScript Output:
```javascript
function multiply(x, y) {
  if (typeof x === "number") {
    return x * y;
  }
  return x.repeat(y);
}
```

### Compiler Resolution Order

When you call an overloaded function, TypeScript checks the overload signatures **top-to-bottom** in the order they are declared:

1. It attempts to match argument types with **Overload 1**.
2. If it matches, the compiler selects that overload's return type.
3. If not, it moves to **Overload 2**, then **Overload 3**, and so forth.
4. If no signature matches, a compile-time error is thrown.

> **Crucial Rule:** Always place your most specific overload signatures at the top, and broader/more general overload signatures at the bottom!

---

## 4. Overloading Types & Basic Examples

### Type 1: Differing Parameter Types

Overloading based on changing parameter types while maintaining argument count.

```typescript
// Overloads
function parseValue(input: string): number;
function parseValue(input: number[]): number;

// Implementation
function parseValue(input: string | number[]): number {
  if (typeof input === "string") {
    return parseFloat(input);
  }
  return input.reduce((acc, curr) => acc + curr, 0);
}

const num = parseValue("123.45"); // Type: number
const sum = parseValue([1, 2, 3]); // Type: number
```

### Type 2: Differing Parameter Counts (Arity)

Overloading based on the number of arguments passed.

```typescript
// Overloads
function createDate(timestamp: number): Date;
function createDate(year: number, month: number, day: number): Date;

// Implementation
function createDate(
  yearOrTimestamp: number,
  month?: number,
  day?: number
): Date {
  if (month !== undefined && day !== undefined) {
    return new Date(yearOrTimestamp, month - 1, day);
  }
  return new Date(yearOrTimestamp);
}

const d1 = createDate(1700000000000);       // Valid
const d2 = createDate(2026, 10, 24);         // Valid
// const d3 = createDate(2026, 10);          // ❌ Error: No overload expects 2 arguments
```

### Type 3: Dependent Return Types

Changing the return type dynamically based on input values or flags.

```typescript
// Overloads
function getData(id: string, async: true): Promise<{ id: string; data: string }>;
function getData(id: string, async: false): { id: string; data: string };

// Implementation
function getData(
  id: string,
  async: boolean
): Promise<{ id: string; data: string }> | { id: string; data: string } {
  const result = { id, data: "Sample Content" };
  
  if (async) {
    return Promise.resolve(result);
  }
  return result;
}

const syncData = getData("101", false);  // Type: { id: string; data: string }
const asyncData = getData("102", true);  // Type: Promise<{ id: string; data: string }>
```

---

## 5. Overloading with Optional & Default Parameters

Mixing optional parameters with overloads requires careful setup in the implementation signature.

### Example: Searching with Optional Filters and Pagination

```typescript
type User = { id: string; name: string; role: string };

// Overload 1: Fetch all users by role
function findUsers(role: string): User[];

// Overload 2: Fetch users by role with explicit pagination
function findUsers(role: string, limit: number, offset: number): User[];

// Implementation signature with optional parameters
function findUsers(
  role: string,
  limit?: number,
  offset?: number
): User[] {
  const actualLimit = limit ?? 10;
  const actualOffset = offset ?? 0;

  console.log(`Searching role=${role}, limit=${actualLimit}, offset=${actualOffset}`);
  return [{ id: "usr_1", name: "Alice", role }];
}

// Invocation
const r1 = findUsers("admin");             // Allowed (Overload 1)
const r2 = findUsers("editor", 20, 40);    // Allowed (Overload 2)
// const r3 = findUsers("editor", 20);      // ❌ Error: Requires either 1 or 3 arguments
```

---

## 6. Complex Real-World Scenarios

### Scenario 1: Heavily Typed Polymorphic Query Builder / API Client

A database layer query runner that supports single entity lookups vs. batch entity lookups.

```typescript
interface Entity {
  id: number;
  title: string;
}

// Overload 1: Single ID returns single entity
function fetchEntity(id: number): Promise<Entity>;

// Overload 2: Array of IDs returns array of entities
function fetchEntity(ids: number[]): Promise<Entity[]>;

// Overload 3: Query configuration object returning custom paginated output
function fetchEntity(config: { limit: number; offset: number }): Promise<{ items: Entity[]; total: number }>;

// Shared Implementation
async function fetchEntity(
  param: number | number[] | { limit: number; offset: number }
): Promise<Entity | Entity[] | { items: Entity[]; total: number }> {
  if (typeof param === "number") {
    // Single ID lookup logic
    return { id: param, title: `Item ${param}` };
  }

  if (Array.isArray(param)) {
    // Array lookup logic
    return param.map((id) => ({ id, title: `Item ${id}` }));
  }

  // Configuration object logic
  return {
    items: [
      { id: 1, title: "Item 1" },
      { id: 2, title: "Item 2" },
    ],
    total: 100,
  };
}

// Usage:
async function runQuery() {
  const single = await fetchEntity(5);                     // Type: Entity
  const list = await fetchEntity([1, 2, 3]);               // Type: Entity[]
  const page = await fetchEntity({ limit: 10, offset: 0 });  // Type: { items: Entity[]; total: number }
}
```

### Scenario 2: Event Listener Registration (DOM Style)

An Event Emitter pattern mapping string event names to specific typed payload callbacks.

```typescript
interface EventMap {
  click: { x: number; y: number };
  keypress: { key: string; code: number };
  hover: { elementId: string };
}

class TypedEventEmitter {
  private listeners: Map<string, Function[]> = new Map();

  // Overload 1: 'click' event listener
  on(event: "click", listener: (payload: EventMap["click"]) => void): void;
  // Overload 2: 'keypress' event listener
  on(event: "keypress", listener: (payload: EventMap["keypress"]) => void): void;
  // Overload 3: 'hover' event listener
  on(event: "hover", listener: (payload: EventMap["hover"]) => void): void;

  // Implementation Signature
  on(event: string, listener: (payload: any) => void): void {
    if (!this.listeners.has(event)) {
      this.listeners.set(event, []);
    }
    this.listeners.get(event)!.push(listener);
  }

  // Trigger event
  emit<K extends keyof EventMap>(event: K, payload: EventMap[K]): void {
    const callbacks = this.listeners.get(event) || [];
    callbacks.forEach((cb) => cb(payload));
  }
}

// Usage
const emitter = new TypedEventEmitter();

// TypeScript automatically infers the correct payload type!
emitter.on("click", (e) => {
  console.log(`Clicked at X: ${e.x}, Y: ${e.y}`);
});

emitter.on("keypress", (e) => {
  console.log(`Pressed key: ${e.key}`);
});

emitter.emit("click", { x: 100, y: 200 });
```

---

## 7. Summary & Best Practices

| Rule / Tip | Reason / Explanation |
| :--- | :--- |
| **Order Matters** | Place specific overload signatures **above** generic ones. TypeScript evaluates top-to-bottom. |
| **Use Union Types When Possible** | If overloads only differ by parameter types and return the same type, prefer union types (`x: string \| number`) over function overloading. |
| **Implementation Signature is Hidden** | Callers cannot invoke the implementation signature directly if it doesn't match an explicit overload. |
| **Strict Return Typing** | Ensure the implementation signature's return type encompasses all return types specified in the overload signatures. |

```
    Prefer Unions for simple parameter variations:
    ❌ Overkill:
       function log(msg: string): void;
       function log(msg: number): void;

    ✅ Clean:
       function log(msg: string | number): void;
```