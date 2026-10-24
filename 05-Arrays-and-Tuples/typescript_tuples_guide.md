# TypeScript Tuples: Operations, Looping, Patterns & Practice Problems

A comprehensive guide to understanding, manipulating, and using **Tuples** in TypeScript. Learn how tuples differ from standard arrays, how to loop through them safely, key design patterns, and solution walkthroughs for practical problems.

---

## Table of Contents
1. [Understanding Tuples vs Arrays](#1-understanding-tuples-vs-arrays)
2. [Tuple Types and Syntax](#2-tuple-types-and-syntax)
3. [Tuple Operations & Manipulation](#3-tuple-operations--manipulation)
4. [Looping over Tuples](#4-looping-over-tuples)
5. [Common Tuple Design Patterns](#5-common-tuple-design-patterns)
6. [Practice Problem Statements & Solutions](#6-practice-problem-statements--solutions)

---

## 1. Understanding Tuples vs Arrays

In TypeScript, both arrays and tuples use the underlying JavaScript `Array` structure. However, type checking behaves differently:

| Feature | TypeScript Array | TypeScript Tuple |
| :--- | :--- | :--- |
| **Element Types** | Usually homogeneous (e.g., `number[]`) | Heterogeneous with defined order (e.g., `[string, number]`) |
| **Length** | Dynamic / Variable length | Fixed number of elements (unless optional/rest elements used) |
| **Position Meaning**| Index represents sequential position | Index represents specific semantic meaning |

---

## 2. Tuple Types and Syntax

### Basic Tuples
```typescript
// Define a tuple representing [Name, Age, IsActive]
type UserRecord = [string, number, boolean];

const user: UserRecord = ["Alice", 30, true];

// Accessing elements (TypeScript infers accurate types per index)
const username: string = user[0];
const age: number = user[1];
```

### Named Tuple Elements
Named elements improve code readability while retaining fixed types.
```typescript
type Point2D = [x: number, y: number];
type HttpResponse = [statusCode: number, statusMessage: string];

const response: HttpResponse = [200, "OK"];
```

### Readonly Tuples
By default, JavaScript array methods like `.push()` can bypass tuple length checks at runtime. Use `readonly` to prevent mutation.
```typescript
type ReadonlyPoint = readonly [number, number];

const point: ReadonlyPoint = [10, 20];
// point.push(30); // Error: Property 'push' does not exist on type 'readonly [number, number]'.
```

### Optional & Rest Elements
```typescript
// Optional element with ?
type RGBWithAlpha = [number, number, number, number?];
const rgb: RGBWithAlpha = [255, 0, 128]; 

// Rest element tuple
type StringAndNumbers = [string, ...number[]];
const scoreReport: StringAndNumbers = ["Math", 90, 85, 92, 100];
```

---

## 3. Tuple Operations & Manipulation

### Destructuring
```typescript
type Coordinate = [number, number, number];
const position: Coordinate = [12.5, 45.1, 100.0];

// Destructure values with correct inferred types
const [latitude, longitude, altitude] = position;
```

### Spreading Tuples
```typescript
type Pair<T> = [T, T];

const pairA: Pair<number> = [1, 2];
const pairB: Pair<number> = [3, 4];

// Combine into a 4-element tuple
const combined: [number, number, number, number] = [...pairA, ...pairB];
```

### Converting Tuples to Union Types
```typescript
type StatusTuple = [200, 404, 500];
type AllowedStatus = StatusTuple[number]; // Union: 200 | 404 | 500
```

---

## 4. Looping over Tuples

Because tuples are standard JavaScript arrays under the hood, standard array loops apply. However, TypeScript preserves fixed-length typing when working directly with indices or tuple destruction.

### Method 1: Destructured `for...of` Loop
Useful when processing an array of tuples.
```typescript
type KeyValuePair = [string, number];

const entries: KeyValuePair[] = [
  ["apples", 5],
  ["bananas", 12],
  ["cherries", 8]
];

for (const [fruit, count] of entries) {
  console.log(`Fruit: ${fruit}, Count: ${count}`);
}
```

### Method 2: Fixed Bounds Index Loop
When iterating a single heterogeneous tuple, indexed looping can require type narrowing.
```typescript
type MixedTuple = [string, number, boolean];
const data: MixedTuple = ["ID-101", 42, false];

for (let i = 0; i < data.length; i++) {
  const element = data[i]; // Type is string | number | boolean
  
  if (typeof element === "string") {
    console.log(`String value: ${element.toUpperCase()}`);
  } else if (typeof element === "number") {
    console.log(`Number value: ${element.toFixed(2)}`);
  }
}
```

---

## 5. Common Tuple Design Patterns

### Pattern A: Multiple Return Values
Instead of wrapping outputs in custom objects, functions can return tuples.
```typescript
function useToggle(initial: boolean): [boolean, () => void] {
  let state = initial;
  const toggle = () => { state = !state; };
  return [state, toggle];
}

const [isOpen, toggleOpen] = useToggle(false);
```

### Pattern B: Strongly Typed Matrix / Coordinate Systems
```typescript
type Matrix2x2 = [
  [number, number],
  [number, number]
];

const identityMatrix: Matrix2x2 = [
  [1, 0],
  [0, 1]
];
```

### Pattern C: Type-Safe Function Arguments using Rest Parameters
```typescript
type LogArgs = [level: "INFO" | "WARN" | "ERROR", message: string, timestamp: Date];

function logMessage(...args: LogArgs): void {
  const [level, message, timestamp] = args;
  console.log(`[${timestamp.toISOString()}] [${level}]: ${message}`);
}

logMessage("INFO", "Server started", new Date());
```

---

## 6. Practice Problem Statements & Solutions

### Problem 1: RGB to Hex Color Converter

**Problem Statement:**
Create a function `rgbToHex` that accepts an RGB color represented as a tuple `[red: number, green: number, blue: number]`. The function should return the hexadecimal color string (e.g., `#FF0000`). Validate that each color channel is between `0` and `255`. Return `null` if invalid.

**Solution:**
```typescript
type RGB = [red: number, green: number, blue: number];

function rgbToHex(color: RGB): string | null {
  const [r, g, b] = color;

  // Validation
  const isValid = (n: number) => n >= 0 && n <= 255 && Number.isInteger(n);
  if (!isValid(r) || !isValid(g) || !isValid(b)) {
    return null;
  }

  const toHex = (n: number): string => n.toString(16).padStart(2, "0").toUpperCase();

  return `#${toHex(r)}${toHex(g)}${toHex(b)}`;
}

// Example Usage:
console.log(rgbToHex([255, 99, 71])); // "#FF6347"
console.log(rgbToHex([300, 0, 0]));   // null
```

---

### Problem 2: Swapping Coordinates (2D Point Inversion)

**Problem Statement:**
Write a generic function `swapPair` that takes a 2-element tuple `[A, B]` and returns a new tuple with the elements swapped `[B, A]`.

**Solution:**
```typescript
function swapPair<T, U>(pair: [T, U]): [U, T] {
  return [pair[1], pair[0]];
}

// Example Usage:
const original: [string, number] = ["Width", 1080];
const swapped = swapPair(original); // Type: [number, string]

console.log(swapped); // [1080, "Width"]
```

---

### Problem 3: Zip Arrays into Tuple Entries

**Problem Statement:**
Write a function `zip` that takes two arrays of equal length—`keys` and `values`—and combines them into an array of key-value tuples `[K, V][]`. If the arrays differ in length, return `null`.

**Solution:**
```typescript
function zip<K, V>(keys: K[], values: V[]): [K, V][] | null {
  if (keys.length !== values.length) {
    return null;
  }

  const result: [K, V][] = [];
  for (let i = 0; i < keys.length; i++) {
    result.push([keys[i], values[i]]);
  }

  return result;
}

// Example Usage:
const keys = ["id", "name", "role"];
const values = [101, "Alex", "Admin"];

const zipped = zip(keys, values);
console.log(zipped);
// Output: [["id", 101], ["name", "Alex"], ["role", "Admin"]]
```

---

### Problem 4: Range Extrema Calculator

**Problem Statement:**
Write a function `findExtrema` that receives an array of numbers and returns a fixed-length tuple `[min: number, max: number]`. Return `null` if the input array is empty.

**Solution:**
```typescript
type Extrema = [min: number, max: number];

function findExtrema(numbers: number[]): Extrema | null {
  if (numbers.length === 0) return null;

  let min = numbers[0];
  let max = numbers[0];

  for (let i = 1; i < numbers.length; i++) {
    const current = numbers[i];
    if (current < min) min = current;
    if (current > max) max = current;
  }

  return [min, max];
}

// Example Usage:
const scores = [45, 88, 12, 99, 63];
const extrema = findExtrema(scores);

if (extrema) {
  const [minScore, maxScore] = extrema;
  console.log(`Min: ${minScore}, Max: ${maxScore}`); // Min: 12, Max: 99
}
```