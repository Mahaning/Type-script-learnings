# TypeScript Arrays (Part 2): Iteration & Function Operations

Welcome to **Part 2** of our TypeScript Arrays series. After exploring the fundamentals and basic methods in Part 1, this guide focuses on **looping over array elements** and **integrating arrays with functions** using strong typing.

---

## Table of Contents

1. [Looping & Iterating Array Elements](#1-looping--iterating-array-elements)
   * [Traditional `for` Loop](#a-traditional-for-loop)
   * [Using `for...in` Loop](#b-using-forin-loop)
   * [Using `for...of` Loop](#c-using-forof-loop)
   * [Comparing Loops](#d-comparing-loop-mechanisms)
2. [Passing Arrays to Functions](#2-passing-arrays-to-functions)
   * [Basic Array Parameters](#a-basic-array-parameters)
   * [Readonly Array Parameters](#b-readonly-array-parameters)
   * [Rest Parameters (`...args`)](#c-rest-parameters-args)
3. [Functions Returning Arrays](#3-functions-returning-arrays)
   * [Basic Array Returns](#a-basic-array-returns)
   * [Generic Functions Returning Arrays](#b-generic-functions-returning-arrays)
4. [Practical Programming Examples](#4-practical-programming-examples)
5. [Summary](#5-summary)

---

## 1. Looping & Iterating Array Elements

TypeScript provides several ways to iterate through array elements. Selecting the right loop depends on whether you need element values, indices, or custom execution control (e.g., `break` / `continue`).

### A. Traditional `for` Loop

The classic standard `for` loop provides complete control over index traversal, step incrementing, and directional iteration (forward or backward).

```typescript
const scores: number[] = [85, 92, 78, 90, 88];

// Iterating forward
for (let i = 0; i < scores.length; i++) {
  console.log(`Index ${i}: Score = ${scores[i]}`);
}

// Iterating backward
for (let i = scores.length - 1; i >= 0; i--) {
  console.log(`Reverse Index ${i}: Score = ${scores[i]}`);
}
```

**Pros:** Provides explicit index access, step modification (e.g., `i += 2`), and full performance optimization.  
**Cons:** Verbose syntax and susceptible to off-by-one errors.

---

### B. Using `for...in` Loop

The `for...in` loop iterates over the **keys (indices)** of an array as **strings**.

```typescript
const fruits: string[] = ["Apple", "Banana", "Cherry"];

for (const index in fruits) {
  // Note: typeof index is 'string', not 'number'
  console.log(`Index (${typeof index}) ${index}: ${fruits[index]}`);
}
```

> ⚠️ **Important Caution for `for...in`:**
> `for...in` is designed for iterating over **object properties**, not array elements. It treats array indices as string keys (`"0"`, `"1"`), and if custom properties are added to the array object, `for...in` will iterate over those as well. Avoid using `for...in` for standard array iteration.

---

### C. Using `for...of` Loop

The `for...of` loop is the modern standard (introduced in ES6) for iterating directly over array **values**.

```typescript
const users: string[] = ["Alice", "Bob", "Charlie"];

for (const user of users) {
  console.log(`User name: ${user}`);
}

// Accessing both index and element using entries()
for (const [index, user] of users.entries()) {
  console.log(`[${index}] ${user}`);
}
```

**Pros:** Clean, readable syntax; directly accesses element values; supports `break` and `continue`.

---

### D. Comparing Loop Mechanisms

| Loop | Iterates Over | Primary Usage | Supports `break`/`continue`? |
| :--- | :--- | :--- | :---: |
| **Traditional `for`** | Numeric indices | Complex indexing or reverse traversal | Yes |
| **`for...of`** | Values | Clean, modern value-based iteration | Yes |
| **`for...in`** | String keys / properties | Object key iteration (Avoid for arrays) | Yes |
| **`Array.prototype.forEach`** | Values & indices | Functional processing | **No** |

---

## 2. Passing Arrays to Functions

Arrays are passed by **reference** in JavaScript and TypeScript. Defining explicit type signatures ensures that functions receive correctly structured array arguments.

### A. Basic Array Parameters

```typescript
function calculateTotal(prices: number[]): number {
  return prices.reduce((sum, price) => sum + price, 0);
}

const itemPrices: number[] = [19.99, 5.50, 42.00];
const total = calculateTotal(itemPrices); // 67.49
```

### B. Readonly Array Parameters

To prevent a function from mutating the passed array (such as calling `.push()`, `.pop()`, or modifying indices), mark the parameter as `readonly`.

```typescript
function printList(items: readonly string[]): void {
  for (const item of items) {
    console.log(`- ${item}`);
  }

  // ❌ Compile Error: Readonly prevents mutation!
  // items.push("New Item"); 
}
```

### C. Rest Parameters (`...args`)

Rest parameters allow functions to collect multiple individual arguments into a single typed array parameter.

```typescript
function buildSentence(delimiter: string, ...words: string[]): string {
  return words.join(delimiter);
}

// 'words' receives ["TypeScript", "is", "awesome"] as string[]
const sentence = buildSentence(" ", "TypeScript", "is", "awesome");
console.log(sentence); // "TypeScript is awesome"
```

---

## 3. Functions Returning Arrays

Functions can generate, transform, or filter data and return typed arrays.

### A. Basic Array Returns

```typescript
interface Student {
  id: number;
  name: string;
  grade: number;
}

function getHonorRoll(students: Student[], minGrade: number): Student[] {
  return students.filter((student) => student.grade >= minGrade);
}

const classRoster: Student[] = [
  { id: 1, name: "Alice", grade: 95 },
  { id: 2, name: "Bob", grade: 72 },
  { id: 3, name: "Charlie", grade: 88 }
];

const honorStudents: Student[] = getHonorRoll(classRoster, 85);
// [{ id: 1, name: "Alice", grade: 95 }, { id: 3, name: "Charlie", grade: 88 }]
```

### B. Generic Functions Returning Arrays

Generics allow building reusable utility functions that preserve type information for any element type `T`.

```typescript
// Reverses any array without mutating the original
function reverseCopy<T>(items: readonly T[]): T[] {
  return [...items].reverse();
}

const numberList = [1, 2, 3, 4];
const reversedNumbers = reverseCopy(numberList); // Type inferred as number[]

const stringList = ["A", "B", "C"];
const reversedStrings = reverseCopy(stringList); // Type inferred as string[]
```

---

## 4. Practical Programming Examples

### Example 1: Chunking an Array into Sub-arrays

```typescript
function chunkArray<T>(array: T[], size: number): T[][] {
  const chunked: T[][] = [];
  
  for (let i = 0; i < array.length; i += size) {
    const chunk = array.slice(i, i + size);
    chunked.push(chunk);
  }
  
  return chunked;
}

const data = [1, 2, 3, 4, 5, 6, 7];
const chunks = chunkArray(data, 3);
console.log(chunks); // [[1, 2, 3], [4, 5, 6], [7]]
```

### Example 2: Duplicate Removal Engine

```typescript
function removeDuplicates<T>(items: T[]): T[] {
  return Array.from(new Set(items));
}

const mixedTags = ["ts", "js", "ts", "react", "js"];
const uniqueTags = removeDuplicates(mixedTags);
console.log(uniqueTags); // ["ts", "js", "react"]
```

---

## 5. Summary

* **Iteration Selection**: Use `for...of` for clean element iteration; use traditional `for` loops when explicit index arithmetic or step sizes are required.
* **Avoid `for...in`**: Do not use `for...in` for standard array iteration because it iterates over string keys and inherited object properties.
* **Array Parameters**: Use `readonly T[]` in function parameter signatures to guarantee immutability and prevent side-effect bugs.
* **Return Types**: Annotate array return types explicitly (`T[]` or `Array<T>`) or leverage generics (`<T>`) to create type-safe utility functions.