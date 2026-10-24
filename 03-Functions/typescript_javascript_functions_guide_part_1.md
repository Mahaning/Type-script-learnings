# 🧩 JavaScript & TypeScript Functions Guide: Part 1 - Named Functions & Fundamentals

Functions are the primary building blocks in JavaScript and TypeScript. They represent reusable units of executable code that accept inputs, perform operations, and return an output.

---

## 📑 Table of Contents

1. [Definition & Core Concepts](#1-definition--core-concepts)
2. [Taxonomy of Functions in JS/TS](#2-taxonomy-of-functions-in-jsts)
3. [Named Functions: Definition, Syntax & Examples](#3-named-functions-definition-syntax--examples)
4. [Named Functions with Parameters](#4-named-functions-with-parameters)
5. [Named Functions with Rest Parameters](#5-named-functions-with-rest-parameters)
6. [Named Functions with Union Types (TypeScript)](#6-named-functions-with-union-types-typescript)
7. [Optional & Default Parameters](#7-optional--default-parameters)
8. [Internal Working & Engine Execution Mechanics](#8-internal-working--engine-execution-mechanics)

---

## 1. Definition & Core Concepts

In JavaScript, functions are **First-Class Objects** (instances of the `Function` prototype). This means functions can be:
- Assigned to variables or object properties.
- Passed as arguments to other functions (higher-order functions).
- Returned from other functions (closures).
- Attached with custom properties and methods.

```typescript
// Demonstrating First-Class Function behavior
function greet(name: string): string {
  return `Hello, ${name}`;
}

// Function attached to a property
greet.language = "English";

// Function assigned to a variable
const fnRef = greet;
console.log(fnRef("Alice")); // "Hello, Alice"
console.log(fnRef.language); // "English"
```

---

## 2. Taxonomy of Functions in JS/TS

JavaScript and TypeScript support several syntactic paradigms for defining functions:

| Function Type | Syntax Syntax Example | Key Characteristic |
| :--- | :--- | :--- |
| **Named Function / Declaration** | `function calculate() {}` | Fully hoisted, bound `this` context, `arguments` object available. |
| **Anonymous / Expression** | `const fn = function() {}` | Variable hoisted (as `undefined`), function expression evaluated at runtime. |
| **Arrow Function** | `const fn = () => {}` | Lexical `this`, no `arguments` object, cannot be used as constructors. |
| **Method Definition** | `const obj = { run() {} }` | Concise method syntax inside objects/classes. |
| **Constructor Function** | `function User(name) { this.name = name; }` | Invoked with `new`, creates instance objects via prototype chain. |
| **Generator Function** | `function* gen() { yield 1; }` | Returns an `IterableIterator`, supports pausing via `yield`. |
| **Async Function** | `async function fetchData() {}` | Returns a `Promise`, enables `await` inside execution context. |

---

## 3. Named Functions: Definition, Syntax & Examples

### Definition
A **Named Function (Function Declaration)** is a function declared with the `function` keyword followed by a mandatory identifier (function name). 

### Syntax
```typescript
function functionName(param1: Type1, param2: Type2): ReturnType {
  // Function body
  return value;
}
```

### Key Feature: Function Hoisting
Named function declarations are **hoisted** by the engine during the compilation phase. This allows a named function to be invoked *before* its definition line in the source code.

```typescript
// Invocation before definition works due to hoisting
const sum = addNumbers(10, 20); 
console.log(`Sum: ${sum}`); // Output: Sum: 30

function addNumbers(a: number, b: number): number {
  return a + b;
}
```

---

## 4. Named Functions with Parameters

Parameters act as local variable placeholders inside the function scope, populated by argument values passed at invocation.

### Typed Parameters and Return Type
In TypeScript, parameters and return values can be explicitly typed to enforce type safety at compile-time.

```typescript
function calculateRectangleArea(width: number, height: number): number {
  return width * height;
}

const area = calculateRectangleArea(5, 8); // 40
// calculateRectangleArea("5", 8); // ❌ TypeScript Compiler Error
```

---

## 5. Named Functions with Rest Parameters

### Definition
A **Rest Parameter** allows a function to accept an indefinite number of arguments as an array using the spread operator (`...`).

### Rules
1. A function can have **only one** rest parameter.
2. The rest parameter **must be the last parameter** in the function signature.

```typescript
function aggregateScores(studentName: string, ...scores: number[]): string {
  const totalScore = scores.reduce((acc, score) => acc + score, 0);
  const average = scores.length > 0 ? totalScore / scores.length : 0;
  
  return `${studentName} achieved an average score of ${average.toFixed(1)}.`;
}

console.log(aggregateScores("Alex", 85, 90, 92, 78)); 
// Output: "Alex achieved an average score of 86.2."
```

---

## 6. Named Functions with Union Types (TypeScript)

Union types (`TypeA | TypeB`) allow named functions to accept arguments of multiple types or return multiple possible types. Within the function, **Type Narrowing** is used to handle each type safely.

```typescript
function formatInput(input: string | number | boolean): string {
  // Type Narrowing using typeof
  if (typeof input === "string") {
    return `[String]: ${input.toUpperCase()}`;
  } else if (typeof input === "number") {
    return `[Number]: $${input.toFixed(2)}`;
  } else {
    return `[Boolean]: ${input ? "ENABLED" : "DISABLED"}`;
  }
}

console.log(formatInput("hello")); // "[String]: HELLO"
console.log(formatInput(49.99));   // "[Number]: $49.99"
console.log(formatInput(true));    // "[Boolean]: ENABLED"
```

---

## 7. Optional & Default Parameters

### A. Optional Parameters (`param?: Type`)
Optional parameters permit arguments to be omitted during invocation. They default to `undefined` if omitted.
*Rule:* Optional parameters must appear **after** required parameters.

### B. Default Parameters (`param: Type = defaultValue`)
Default parameters specify a fallback value used if the argument is omitted or passed explicitly as `undefined`.

```typescript
// Combining Required, Default, and Optional Parameters
function createHTTPConnection(
  host: string,                      // Required
  port: number = 8080,               // Default Parameter
  timeoutMs?: number                 // Optional Parameter (number | undefined)
): string {
  let details = `Connecting to ${host}:${port}`;
  if (timeoutMs !== undefined) {
    details += ` (Timeout: ${timeoutMs}ms)`;
  }
  return details;
}

console.log(createHTTPConnection("localhost")); 
// Output: "Connecting to localhost:8080"

console.log(createHTTPConnection("192.168.1.1", 3000)); 
// Output: "Connecting to 192.168.1.1:3000"

console.log(createHTTPConnection("api.domain.com", undefined, 5000)); 
// Output: "Connecting to api.domain.com:8080 (Timeout: 5000ms)"
```

---

## 8. Internal Working & Engine Execution Mechanics

When JavaScript engines (such as V8) compile and run a Named Function, several steps occur across memory allocation and runtime context creation:

```
    ┌─────────────────────────────────────────────────────────────┐
    │ 1. Compilation Phase (Creation Phase)                       │
    │    • Function Identifier is registered in Memory Space.     │
    │    • Full function code block is hoisted to variable scope.  │
    └──────────────────────────────┬──────────────────────────────┘
                                   │
                                   ▼
    ┌─────────────────────────────────────────────────────────────┐
    │ 2. Invocation Phase (Execution Phase)                       │
    │    • A new Function Execution Context (FEC) is pushed onto   │
    │      the Call Stack.                                        │
    └──────────────────────────────┬──────────────────────────────┘
                                   │
                                   ▼
    ┌─────────────────────────────────────────────────────────────┐
    │ 3. Internal Context Setup                                   │
    │    • Creates Variable Environment (Parameters & Locals).    │
    │    • Binds 'this' binding.                                  │
    │    • Instantiates 'arguments' object (Array-like parameter).│
    └──────────────────────────────┬──────────────────────────────┘
                                   │
                                   ▼
    ┌─────────────────────────────────────────────────────────────┐
    │ 4. Execution & Stack Cleanup                                │
    │    • Bytecode/Native machine code is executed.              │
    │    • Return value pushed to caller.                         │
    │    • FEC is popped off Call Stack (Garbage collected if no │
    │      closure references exist).                             │
    └─────────────────────────────────────────────────────────────┘
```

### Memory & Call Stack Breakdown

1. **Heap Allocation:** Function objects are allocated in Heap Memory. The variable name in the Scope holds a pointer/reference to this Heap object.
2. **Arguments Allocation:**
   - **Standard & Rest parameters:** Arguments are placed directly into local activation registers/slots.
   - Rest parameters (`...args`) trigger the instantiation of an actual JavaScript `Array` instance on the Heap during call setup.
3. **Optimizations (Hidden Classes & Inline Caching):**
   - V8 creates **Feedback Vectors** for functions that are called repeatedly ("hot functions").
   - If parameter types remain consistent across invocations, TurboFan JIT-compiles the function bytecode into optimized machine code, skipping repeated runtime type checks.