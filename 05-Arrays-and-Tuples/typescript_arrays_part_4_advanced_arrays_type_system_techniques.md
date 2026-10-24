# TypeScript Arrays (Part 4): Advanced Arrays & Type System Techniques

Welcome to **Part 4** of our TypeScript Arrays series. Building upon fundamentals, iteration mechanics, V8 performance, and multidimensional structures, this guide explores **advanced TypeScript array techniques**, focusing on the type system, immutable patterns, custom array subclasses, and high-performance array operations.

---

## Table of Contents

1. [Tuples vs. Arrays](#1-tuples-vs-arrays)
2. [Variadic Tuple Types & Rest Elements](#2-variadic-tuple-types--rest-elements)
3. [Type Guards & Predicates with Arrays](#3-type-guards--predicates-with-arrays)
4. [Mapped & Conditional Types on Arrays](#4-mapped--conditional-types-on-arrays)
5. [Custom Array Subclasses](#5-custom-array-subclasses)
6. [Advanced Functional Operations (Zip, Partition, GroupBy)](#6-advanced-functional-operations-zip-partition-groupby)
7. [Typed Arrays & Low-Level ArrayBuffer Performance](#7-typed-arrays--low-level-arraybuffer-performance)
8. [Summary](#8-summary)

---

## 1. Tuples vs. Arrays

While standard arrays represent collections of variable length with elements of uniform or union types, **Tuples** represent arrays with a **fixed number of elements** where each position has a specific type.

```typescript
// Standard Array: Flexible length, homogeneous or union elements
type StringOrNumberArray = (string | number)[];
const arr: StringOrNumberArray = ["Alice", 42, "Bob"];

// Tuple: Fixed length, strictly ordered position types
type UserRecord = [id: number, name: string, isActive: boolean];
const user: UserRecord = [101, "Alice", true];

// Labeled Tuples enhance clarity in editor autocomplete:
type Point3D = [x: number, y: number, z: number];
```

### Const Assertions for Immutable Literal Tuples

Using `as const` creates read-only tuples with literal types rather than primitive widenings.

```typescript
// Without 'as const': Type inferred as string[]
const HTTP_METHODS_ARR = ["GET", "POST", "PUT", "DELETE"]; 

// With 'as const': Type inferred as readonly ["GET", "POST", "PUT", "DELETE"]
const HTTP_METHODS = ["GET", "POST", "PUT", "DELETE"] as const;

type HttpMethod = typeof HTTP_METHODS[number]; 
// Inferred Union Type: "GET" | "POST" | "PUT" | "DELETE"
```

---

## 2. Variadic Tuple Types & Rest Elements

TypeScript allows rest elements `...T` inside tuple signatures, enabling flexible tuple transformations and function argument modeling.

```typescript
// Rest element at the end
type StringAndNumbers = [string, ...number[]];
const data1: StringAndNumbers = ["Total", 10, 20, 30];

// Rest element at the start or middle
type NamedTail<T extends any[]> = [name: string, ...T];
type NumberTail = NamedTail<[number, boolean]>; 
// Result: [name: string, number, boolean]

// Concatenating Tuple Types
type Concat<T extends readonly any[], U extends readonly any[]> = [...T, ...U];
type Combined = Concat<[1, 2], ["A", "B"]>; 
// Result: [1, 2, "A", "B"]
```

---

## 3. Type Guards & Predicates with Arrays

Filtering arrays often results in union types containing `undefined` or `null`. Using custom **Type Predicates** (`item is T`) narrows array types cleanly.

```typescript
const mixedValues: (string | null | undefined)[] = ["Alice", null, "Bob", undefined, "Charlie"];

// ❌ Standard filter retains null & undefined in the return type
const badFilter = mixedValues.filter(x => x !== null && x !== undefined);
// Type: (string | null | undefined)[]

// ✅ Custom Type Guard function
function isNotNullOrUndefined<T>(value: T | null | undefined): value is T {
  return value !== null && value !== undefined;
}

const cleanValues: string[] = mixedValues.filter(isNotNullOrUndefined);
// Type: string[]
```

---

## 4. Mapped & Conditional Types on Arrays

You can extract or transform array types dynamically using conditional types and the `infer` keyword.

```typescript
// Extract the element type of any Array or Tuple
type ElementOf<T> = T extends readonly (infer E)[] ? E : never;

type NumberType = ElementOf<number[]>;       // number
type TupleElem  = ElementOf<[string, boolean]>; // string | boolean

// Convert a Tuple of Promises into a Tuple of Resolved Types
type AwaitedTuple<T extends readonly any[]> = {
  [K in keyof T]: T[K] extends Promise<infer R> ? R : T[K];
};

type Promises = [Promise<number>, Promise<string>, boolean];
type Resolved = AwaitedTuple<Promises>; 
// Result: [number, string, boolean]
```

---

## 5. Custom Array Subclasses

You can extend built-in JavaScript `Array` in TypeScript to build rich domain models or queryable collections.

```typescript
class EntityCollection<T extends { id: number }> extends Array<T> {
  findById(id: number): T | undefined {
    return this.find(item => item.id === id);
  }

  removeById(id: number): boolean {
    const index = this.findIndex(item => item.id === id);
    if (index !== -1) {
      this.splice(index, 1);
      return true;
    }
    return false;
  }
}

// Usage
interface User {
  id: number;
  name: string;
}

const users = new EntityCollection<User>();
users.push({ id: 1, name: "Alice" }, { id: 2, name: "Bob" });

const user1 = users.findById(1); // { id: 1, name: "Alice" }
users.removeById(2);
console.log(users.length); // 1
```

---

## 6. Advanced Functional Operations (Zip, Partition, GroupBy)

TypeScript signatures for advanced array utilities maintain strong type inference across inputs and outputs.

### A. Partition Utility (Split by condition)

```typescript
function partition<T>(
  array: T[],
  predicate: (item: T) => boolean
): [pass: T[], fail: T[]] {
  const pass: T[] = [];
  const fail: T[] = [];
  for (const item of array) {
    if (predicate(item)) {
      pass.push(item);
    } else {
      fail.push(item);
    }
  }
  return [pass, fail];
}

const numbers = [1, 2, 3, 4, 5, 6];
const [evens, odds] = partition(numbers, n => n % 2 === 0);
// evens: [2, 4, 6], odds: [1, 3, 5]
```

### B. Zip Utility (Combine two arrays into tuples)

```typescript
function zip<T, U>(arr1: T[], arr2: U[]): [T, U][] {
  const length = Math.min(arr1.length, arr2.length);
  const zipped: [T, U][] = [];
  for (let i = 0; i < length; i++) {
    zipped.push([arr1[i], arr2[i]]);
  }
  return zipped;
}

const keys = ["id", "name", "role"];
const vals = [101, "Alice", "Admin"];
const paired = zip(keys, vals); 
// [["id", 101], ["name", "Alice"], ["role", "Admin"]]
```

### C. GroupBy Utility (`Object.groupBy` equivalent)

```typescript
function groupBy<T, K extends PropertyKey>(
  array: T[],
  getKey: (item: T) => K
): Record<K, T[]> {
  return array.reduce((acc, item) => {
    const key = getKey(item);
    if (!acc[key]) {
      acc[key] = [];
    }
    acc[key].push(item);
    return acc;
  }, {} as Record<K, T[]>);
}

interface InventoryItem { name: string; category: "fruit" | "vegetable" }
const inventory: InventoryItem[] = [
  { name: "Apple", category: "fruit" },
  { name: "Carrot", category: "vegetable" },
  { name: "Banana", category: "fruit" }
];

const grouped = groupBy(inventory, item => item.category);
// { fruit: [{ name: "Apple"... }, { name: "Banana"... }], vegetable: [{ name: "Carrot"... }] }
```

---

## 7. Typed Arrays & Low-Level ArrayBuffer Performance

For high-performance applications (audio processing, graphics, binary data), TypeScript supports native **TypedArrays** operating over raw binary memory (`ArrayBuffer`).

Unlike JS Arrays, TypedArrays have **fixed memory layouts, flat byte structures, and no runtime overhead or element kind checks**.

```typescript
// Allocate a raw binary buffer of 16 bytes
const buffer = new ArrayBuffer(16);

// Create 32-bit Integer view over the buffer (4 bytes per integer -> 4 elements total)
const int32View = new Int32Array(buffer);
int32View[0] = 42;
int32View[1] = 100;

// Create 8-bit Unsigned Integer view over the SAME memory buffer
const uint8View = new Uint8Array(buffer);
console.log(uint8View.length); // 16 bytes

// Float64Array for high-precision vector arithmetic
const vector = new Float64Array([1.5, 3.14159, 9.81]);
```

### Common Typed Arrays in TypeScript

| TypedArray | Element Size | Description |
| --- | --- | --- |
| `Int8Array` / `Uint8Array` | 1 byte | 8-bit signed/unsigned integers |
| `Uint8ClampedArray` | 1 byte | Clamped integers 0-255 (ideal for Canvas pixel data) |
| `Int32Array` / `Uint32Array` | 4 bytes | 32-bit signed/unsigned integers |
| `Float32Array` / `Float64Array` | 4 / 8 bytes | IEEE 754 floating-point numbers |
| `BigInt64Array` | 8 bytes | 64-bit BigInt values |

---

## 8. Summary

* **Tuples vs Arrays**: Use tuples when position dictates semantic meaning and exact length matters; use arrays for collections of dynamic size.
* **Const Assertions**: Use `as const` to convert array literals into immutable tuples with exact literal types.
* **Type Guards**: Use user-defined type predicates (`val is T`) to remove `null` or `undefined` types when filtering array inputs.
* **Type Transformations**: Infer element types (`infer E`) or transform tuple elements mapped type signatures dynamically.
* **Low-Level Performance**: Use `TypedArrays` (`Float32Array`, `Uint8Array`) when working with binary streams, WebGL, or performance-critical numeric pipelines.