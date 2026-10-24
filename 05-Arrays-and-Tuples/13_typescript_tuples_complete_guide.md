# TypeScript Tuples: Comprehensive Guide

A **Tuple** in TypeScript is a special type of array that has a **fixed number of elements**, where **each element has a specific, known type** at a specific position (index). 

Unlike standard TypeScript arrays—which represent homogeneous, dynamic-length collections—tuples are used to group heterogeneous data sets with rigid positional contracts.

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Declaration Syntax](#2-declaration-syntax)
3. [Key Characteristics](#3-key-characteristics)
4. [Primary Use Cases](#4-primary-use-cases)
5. [Accessing Elements](#5-accessing-elements)
6. [Altering & Updating Elements](#6-altering--updating-elements)
7. [Deleting Elements](#7-deleting-elements)
8. [Tuple Operations & Methods](#8-tuple-operations--methods)
9. [Advanced Tuple Types](#9-advanced-tuple-types)
10. [Summary Checklist](#10-summary-checklist)

---

## 1. Introduction

In JavaScript, arrays can hold any type of data and grow dynamically. TypeScript introduces **Tuples** to allow developers to express fixed-structure array representations, similar to structures or records in low-level languages.

```
Standard Array (string | number)[]:  ["Alice", 28, "Bob", 34]  (Any length, mixed order)
Tuple [string, number]:              ["Alice", 28]             (Fixed length, exact order)
```

---

## 2. Declaration Syntax

### Basic Declaration

Define the types inside square brackets `[Type1, Type2, ...]`:

```typescript
// Explicit Type Annotation
let user: [number, string, boolean];

// Initialization
user = [1, "Alice", true]; // ✅ Valid

// ❌ Error: Type order does not match declaration
// user = ["Alice", 1, true]; 

// ❌ Error: Missing required element at index 2
// user = [1, "Alice"]; 
```

### Labeled Tuples (Self-Documenting Syntax)

You can assign descriptive labels to tuple elements. Labels do not alter runtime behavior, but they provide critical documentation in editor tooltips.

```typescript
// Labeled Tuple
type GeoCoordinates = [latitude: number, longitude: number, altitude?: number];

const location: GeoCoordinates = [37.7749, -122.4194];
```

---

## 3. Key Characteristics

1. **Fixed Position & Type Order**: Element at index `0` must match type `0`, index `1` must match type `1`, etc.
2. **Fixed Length (by default)**: TypeScript enforces length limits during element assignment.
3. **Compiles to Standard JavaScript Arrays**: At runtime, tuples compile directly down to standard JavaScript arrays (`[...]`).
4. **Heterogeneous Capabilities**: Designed explicitly to store elements of different types without requiring generic union widening `(string | number)[]`.

---

## 4. Primary Use Cases

1. **Function Multiple Return Values**: Returning multiple named properties without creating full object instances (e.g., React hooks like `useState`).
2. **Coordinate & Vector Data**: Representing spatial vectors, RGB colors, or geographic points (`[x, y, z]`, `[r, g, b]`).
3. **CSV or Database Row Representations**: Mapping database rows directly into positionally ordered items (`[id, email, role]`).

```typescript
// Example: React-style State Hook returning a Tuple
function useCustomState<T>(initialValue: T): [T, (newValue: T) => void] {
  let val = initialValue;
  const setVal = (newValue: T) => { val = newValue; };
  return [val, setVal];
}

const [count, setCount] = useCustomState<number>(0);
```

---

## 5. Accessing Elements

### Direct Indexing

Access elements using standard bracket notation `tuple[index]`. TypeScript infers the exact type corresponding to that position.

```typescript
type Student = [id: number, name: string, gpa: number];

const student: Student = [101, "Sarah", 3.9];

const id = student[0];    // Type: number
const name = student[1];  // Type: string
const gpa = student[2];   // Type: number

// ❌ Compile Error: Index 3 is out of bounds!
// const outOfBounds = student[3]; 
```

### Destructuring

Tuples are commonly unpacked using array destructuring:

```typescript
const [studentId, studentName] = student; 
// studentId: number, studentName: string
```

---

## 6. Altering & Updating Elements

### Mutating Assigned Elements

You can update values at valid tuple indices as long as the replacement matches the specified type for that position.

```typescript
let response: [statusCode: number, statusText: string] = [200, "OK"];

// ✅ Valid update matching index type
response[0] = 404;
response[1] = "Not Found";

// ❌ Compile Error: Cannot assign string to number at index 0
// response[0] = "Error"; 
```

### The Push/Pop Mutation Gotcha (Important!)

Because tuples compile down to plain JavaScript arrays, mutating methods like `.push()`, `.pop()`, `.shift()`, and `.unshift()` exist on tuple instances at runtime.

```typescript
let point: [number, number] = [10, 20];

// ⚠️ TypeScript allows .push() at compile time if value matches union of element types!
point.push(30); 
console.log(point); // Runtime Output: [10, 20, 30] (Length is now 3!)

// However, accessing the pushed element directly triggers a compile error:
// console.log(point[2]); // ❌ Error: Tuple type '[number, number]' of length '2' has no element at index '2'.
```

#### Prevention: Readonly Tuples
To strictly prevent mutation, mark the tuple as `readonly` or use `as const`:

```typescript
// Readonly tuple prevents push, pop, or index mutation
type StrictPoint = readonly [number, number];

const immutablePoint: StrictPoint = [10, 20];

// ❌ All mutation attempts fail at compile time:
// immutablePoint[0] = 15; // Error
// immutablePoint.push(30); // Error: Property 'push' does not exist on type 'readonly [number, number]'
```

---

## 7. Deleting Elements

JavaScript arrays (and by extension TypeScript tuples) do not support deleting element slots without modifying array indices or length.

### 1. `delete` Operator (Creates Sparse Elements)

Using `delete tuple[index]` clears the value and leaves `undefined` in its slot, but preserves array length.

```typescript
let record: [number, string] = [1, "Data"];

delete record[1]; // Sets record[1] to undefined
console.log(record); // [1, <1 empty item>]
```

> **Warning**: Avoid `delete` on tuples because it breaks type safety by introducing `undefined` without updating the type signature.

### 2. Splice / Structural Deletion (Returns New Array)

Removing an element using `.splice()` or `.filter()` returns a general **Array** rather than preserving the original Tuple type structure:

```typescript
let tuple: [string, number, boolean] = ["A", 10, true];

// Removing an element turns the result into a standard array (string | number | boolean)[]
const remaining = tuple.filter((_, index) => index !== 1); 
```

---

## 8. Tuple Operations & Methods

Tuples support all standard `Array.prototype` methods, but their return types adapt based on the operation:

### Non-Mutating Methods

```typescript
const pair: [string, number] = ["Age", 30];

// .concat()
const merged = pair.concat(["Years"]); 
// Return type inferred as (string | number)[]

// .slice()
const sliced = pair.slice(0, 1); 
// Inferred as (string | number)[]

// .map()
const stringified = pair.map(val => String(val)); 
// Inferred as string[]
```

---

## 9. Advanced Tuple Types

### A. Optional Tuple Elements

Mark trailing elements with `?` to make them optional:

```typescript
type Padding = [top: number, right: number, bottom?: number, left?: number];

const pad1: Padding = [10, 20];         // ✅ Valid (Length 2)
const pad2: Padding = [10, 20, 10, 20]; // ✅ Valid (Length 4)
```

### B. Variadic Tuple Types (Rest Elements)

Rest elements (`...Type[]`) allow tuples to accept variable numbers of elements at specified positions.

```typescript
// Rest at the end
type StringAndNumbers = [string, ...number[]];
const data1: StringAndNumbers = ["Score", 100, 95, 88, 92];

// Rest in the middle
type Payload = [id: number, ...tags: string[], isVerified: boolean];
const item: Payload = [101, "ts", "frontend", "web", true];
```

### C. `as const` Assertions

Using `as const` infers literal types and applies `readonly` automatically:

```typescript
const config = ["HTTPS", 443] as const;
// Inferred Type: readonly ["HTTPS", 443]

// ❌ Compile Error
// config[0] = "HTTP"; 
```

---

## 10. Summary Checklist

| Concept | Description | Syntax Example |
| :--- | :--- | :--- |
| **Declaration** | Defines exact types per index position | `let x: [string, number];` |
| **Labeled Syntax** | Adds documentation names to elements | `[name: string, age: number]` |
| **Readonly Tuples** | Prevents `.push()`, `.pop()`, and index mutations | `readonly [number, number]` |
| **Optional Elements** | Trailing elements marked with `?` | `[string, number?]` |
| **Variadic Tuples** | Includes flexible rest parameters | `[string, ...boolean[]]` |
| **Literal Tuples** | Declares fixed values using `as const` | `["GET", 200] as const` |
