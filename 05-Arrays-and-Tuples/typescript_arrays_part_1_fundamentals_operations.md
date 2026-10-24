# TypeScript Arrays (Part 1): Fundamentals & Operations

Welcome to **Part 1** of our TypeScript Arrays series. Arrays are ordered collections of values used to store multiple items of the same or mixed types under a single variable name. TypeScript builds upon JavaScript arrays by adding **strict compile-time type checking**.

---

## Table of Contents

1. [Introduction & Definition](#1-introduction--definition)
2. [Key Characteristics of TypeScript Arrays](#2-key-characteristics-of-typescript-arrays)
3. [Declaring Arrays](#3-declaring-arrays)
   * [Method 1: Array Literal Syntax (`T[]`)](#method-1-array-literal-syntax-t)
   * [Method 2: Generic Array Type (`Array<T>`)](#method-2-generic-array-type-arrayt)
   * [Method 3: Readonly Arrays](#method-3-readonly-arrays)
4. [Array Types](#4-array-types)
   * [Single-type Arrays](#single-type-arrays)
   * [Union Type Arrays](#union-type-arrays)
   * [Multidimensional Arrays](#multidimensional-arrays)
5. [Indexing & Accessing Elements](#5-indexing--accessing-elements)
6. [Essential Array Methods & Functions](#6-essential-array-methods--functions)
   * [Mutating Methods (Modify original array)](#a-mutating-methods)
   * [Non-Mutating Methods (Return a new array/value)](#b-non-mutating-methods)
   * [Search & Inspection Methods](#c-search--inspection-methods)
   * [Iteration Methods](#d-iteration-methods)
7. [Summary](#7-summary)

---

## 1. Introduction & Definition

An **Array** in TypeScript is a high-level, list-like object used to represent an ordered collection of values. 

In pure JavaScript, arrays can hold any combination of data types dynamically. TypeScript enhances JavaScript arrays by allowing you to enforce **type safety**—ensuring that an array only contains elements of specified types.

```
+-------------------------------------------------------------------+
|                        Array Index Structure                      |
+-------------------------------------------------------------------+
|  Index:       [0]          [1]          [2]          [3]          |
|  Value:    "Apple"      "Banana"      "Cherry"      "Date"        |
|  Type:      string       string       string        string        |
+-------------------------------------------------------------------+
```

---

## 2. Key Characteristics of TypeScript Arrays

1. **Zero-Indexed**: The first element of an array is located at index `0`, the second at index `1`, and so on.
2. **Dynamic Sizing**: Arrays in TypeScript/JavaScript can automatically expand or contract as elements are added or removed.
3. **Type-Enforced**: TypeScript enforces that all elements pushed into or contained inside an array match its declared type.
4. **Ordered**: Elements maintain their insertion order unless explicitly re-ordered or sorted.
5. **Heterogeneous vs. Homogeneous**: Arrays can be restricted to a single type (homogeneous) or configured to accept multiple explicit types using Union Types (heterogeneous).

---

## 3. Declaring Arrays

There are two primary syntaxes for declaring arrays in TypeScript, plus read-only variations.

### Method 1: Array Literal Syntax (`T[]`)

This is the most common and preferred syntax. Append square brackets `[]` to the element type `T`.

```typescript
// Explicit type annotation
let numbers: number[] = [10, 20, 30, 40];
let fruits: string[] = ["Apple", "Banana", "Orange"];

// Type Inference (TypeScript infers string[])
let colors = ["Red", "Green", "Blue"];

// Empty array declaration (requires explicit type annotation)
let pendingTasks: string[] = [];
pendingTasks.push("Write documentation");
```

### Method 2: Generic Array Type (`Array<T>`)

Using TypeScript's built-in `Array<T>` generic interface. Functionally identical to `T[]`.

```typescript
let scores: Array<number> = [95, 88, 72, 100];
let userNames: Array<string> = ["Alice", "Bob", "Charlie"];

// Array of objects using generics
interface Product {
  id: number;
  name: string;
}

let inventory: Array<Product> = [
  { id: 1, name: "Laptop" },
  { id: 2, name: "Mouse" }
];
```

### Method 3: Readonly Arrays

Use `ReadonlyArray<T>` or `readonly T[]` to create immutable arrays that cannot be modified after initialization.

```typescript
const originalScores: readonly number[] = [100, 90, 80];
// Or using generic syntax:
// const originalScores: ReadonlyArray<number> = [100, 90, 80];

// ❌ Compile Errors:
// originalScores.push(70); // Error: Property 'push' does not exist on type 'readonly number[]'.
// originalScores[0] = 99;   // Error: Index signature in type 'readonly number[]' only permits reading.
```

---

## 4. Array Types

### Single-type Arrays
Holds values of only one specific primitive or object type.

```typescript
let flags: boolean[] = [true, false, true];
```

### Union Type Arrays
Holds values of multiple specified types using union syntax `(TypeA | TypeB)[]`.

```typescript
// Accepts numbers OR strings
let mixedData: (number | string)[] = [1, "Two", 3, "Four"];

// ❌ Compile Error:
// mixedData.push(true); // Error: Argument of type 'boolean' is not assignable to 'number | string'.
```

### Multidimensional Arrays
Arrays containing other arrays (matrices or grids).

```typescript
// 2D Array (Grid / Matrix)
let matrix: number[][] = [
  [1, 2, 3],
  [4, 5, 6],
  [7, 8, 9]
];

// Accessing 2D Array: row 1, column 2
console.log(matrix[1][2]); // Output: 6

// 3D Array
let cube: number[][][] = [
  [[1, 2], [3, 4]],
  [[5, 6], [7, 8]]
];
```

---

## 5. Indexing & Accessing Elements

Elements are accessed and modified using bracket notation `array[index]`.

```typescript
let cities: string[] = ["Tokyo", "Paris", "New York", "London"];

// Accessing elements
let firstCity = cities[0];  // "Tokyo"
let thirdCity = cities[2];  // "New York"

// Modifying elements
cities[1] = "Berlin"; // Replaces "Paris" with "Berlin"

// Checking length
console.log(cities.length); // Output: 4

// Accessing non-existent index
let unknownCity = cities[99]; // Returns 'undefined' at runtime
```

> **TypeScript Configuration Note (`noUncheckedIndexedAccess`):**  
> If `noUncheckedIndexedAccess: true` is enabled in `tsconfig.json`, TypeScript will infer the type of `cities[index]` as `string | undefined`, forcing safe runtime checks before accessing properties.

---

## 6. Essential Array Methods & Functions

### A. Mutating Methods
These methods alter the original array in place.

| Method | Description | Example |
| :--- | :--- | :--- |
| `push(...items)` | Adds items to the **end** of the array. | `arr.push(40);` |
| `pop()` | Removes and returns the **last** item. | `let last = arr.pop();` |
| `unshift(...items)`| Adds items to the **beginning** of the array. | `arr.unshift(0);` |
| `shift()` | Removes and returns the **first** item. | `let first = arr.shift();` |
| `splice(start, deleteCount, ...items)` | Adds, removes, or replaces elements at any index. | `arr.splice(1, 2, "New");` |
| `reverse()` | Reverses the elements in place. | `arr.reverse();` |
| `sort()` | Sorts the elements in place. | `arr.sort((a, b) => a - b);` |

```typescript
let items: number[] = [10, 20, 30];

items.push(40);      // [10, 20, 30, 40]
items.unshift(5);    // [5, 10, 20, 30, 40]
items.pop();         // [5, 10, 20, 30] -> returns 40
items.shift();       // [10, 20, 30] -> returns 5
```

---

### B. Non-Mutating Methods
These methods do **not** modify the original array; they return a new value or new array instead.

| Method | Description | Example |
| :--- | :--- | :--- |
| `concat(...items)` | Joins two or more arrays together. | `let combined = a1.concat(a2);` |
| `slice(start, end)` | Returns a shallow copy of a portion of the array. | `let sub = arr.slice(1, 3);` |
| `map(callback)` | Returns a new array with results of calling a function on every element. | `let doubled = arr.map(x => x * 2);` |
| `filter(callback)` | Returns a new array with elements that pass a test. | `let evens = arr.filter(x => x % 2 === 0);` |
| `reduce(callback, initialValue)` | Reduces array to a single value via an accumulator. | `let sum = arr.reduce((acc, x) => acc + x, 0);` |
| `flat(depth)` | Flattens nested arrays up to specified depth. | `let flatArr = nested.flat(2);` |

```typescript
const nums = [1, 2, 3, 4, 5];

// Map
const squared = nums.map(n => n * n); // [1, 4, 9, 16, 25]

// Filter
const greaterThanTwo = nums.filter(n => n > 2); // [3, 4, 5]

// Reduce
const totalSum = nums.reduce((acc, curr) => acc + curr, 0); // 15
```

---

### C. Search & Inspection Methods

| Method | Description | Example |
| :--- | :--- | :--- |
| `indexOf(element)` | Returns the first index of an element (-1 if not found). | `arr.indexOf("Apple");` |
| `lastIndexOf(element)`| Returns the last index of an element. | `arr.lastIndexOf("Apple");` |
| `includes(element)`| Returns `true` if array contains specified element. | `arr.includes(10);` |
| `find(predicate)` | Returns the **first element** matching condition. | `arr.find(x => x.id === 1);` |
| `findIndex(predicate)`| Returns index of **first element** matching condition. | `arr.findIndex(x => x > 10);` |
| `some(predicate)` | Returns `true` if **at least one** element matches. | `arr.some(x => x < 0);` |
| `every(predicate)`| Returns `true` if **all** elements match. | `arr.every(x => x > 0);` |

```typescript
const users = [
  { id: 1, name: "Alice" },
  { id: 2, name: "Bob" }
];

const user = users.find(u => u.id === 2); // { id: 2, name: "Bob" }
const hasAlice = users.some(u => u.name === "Alice"); // true
```

---

### D. Iteration Methods

```typescript
const techStack: string[] = ["TypeScript", "React", "Node.js"];

// 1. forEach
techStack.forEach((tech, index) => {
  console.log(`${index}: ${tech}`);
});

// 2. for...of loop
for (const tech of techStack) {
  console.log(tech);
}

// 3. for...in loop (iterates over indices)
for (const index in techStack) {
  console.log(`Index ${index} holds ${techStack[index]}`);
}
```

---

## 7. Summary

* **Array Syntax**: Prefer `T[]` for clean readability, or `Array<T>` for generic consistency.
* **Type Safety**: TypeScript prevents pushing incorrect types into explicitly typed arrays.
* **Read-only Arrays**: Use `readonly T[]` or `ReadonlyArray<T>` to enforce immutability.
* **Method Types**: Always choose between **mutating** (`push`, `pop`, `splice`) and **non-mutating** (`map`, `filter`, `slice`) methods based on whether state mutation is desired.